# Aula 02: Explorando Comandos SLURM

**Objetivo:** Entender como a solicitação de recursos impacta o tempo de execução de uma tarefa e compreender como pedir recursos ao SLURM.

**Recursos Necessários:** Acesso ao cluster Franky configurado.

### Programa C++ para realizar a convolução de uma matriz

Crie um arquivo chamado `convolucao.cpp` com o seguinte conteúdo:

```cpp
#include <iostream>   // Inclui a biblioteca padrão de entrada e saída.
#include <mpi.h>      // Inclui a biblioteca MPI para paralelismo entre múltiplos processos.
#include <omp.h>      // Inclui a biblioteca OpenMP para paralelismo com threads.
#include <chrono>     // Inclui a biblioteca para medição de tempo.

#define N 1000 // Define o tamanho da matriz NxN.
#define FILTER_SIZE 5 // Define o tamanho do filtro 5x5.
#define ITERATIONS 15 // Define o número de iterações de convolução.


// Função para realizar a convolução em um elemento da matriz.
int apply_filter(int x, int y, int matrix[N][N], int filter[FILTER_SIZE][FILTER_SIZE]) {
    int result = 0;
    int filter_offset = FILTER_SIZE / 2;  // Calcula o deslocamento do filtro para centralizar.

    // Aplica o filtro 5x5 ao elemento (x, y) da matriz.
    for (int i = -filter_offset; i <= filter_offset; i++) {
        for (int j = -filter_offset; j <= filter_offset; j++) {
            int xi = x + i;
            int yj = y + j;
            // Verifica se o índice está dentro dos limites da matriz.
            if (xi >= 0 && xi < N && yj >= 0 && yj < N) {
                result += matrix[xi][yj] * filter[i + filter_offset][j + filter_offset];
            }
        }
    }
    return result;  // Retorna o valor convoluído.
}

int main(int argc, char *argv[]) {
    MPI_Init(&argc, &argv);  // Inicializa o ambiente MPI.

    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank); // Obtém o rank (identificador) do processo atual.
    MPI_Comm_size(MPI_COMM_WORLD, &size); // Obtém o número total de processos.

    int matrix[N][N]; // Declara a matriz original.
    int filter[FILTER_SIZE][FILTER_SIZE] = {{1, 1, 1, 1, 1}, 
                                            {1, 1, 1, 1, 1}, 
                                            {1, 1, 1, 1, 1}, 
                                            {1, 1, 1, 1, 1}, 
                                            {1, 1, 1, 1, 1}}; // Define o filtro 5x5 (exemplo de suavização).
    int result[N][N]; // Declara a matriz resultante após a convolução.

    // Inicialização da matriz pelo processo mestre (rank 0).
    if (rank == 0) {
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                matrix[i][j] = i + j; // Exemplo de inicialização: a soma dos índices.
            }
        }
    }

    // Sincroniza todos os processos antes de começar a medir o tempo.
    MPI_Barrier(MPI_COMM_WORLD);
    auto start = std::chrono::high_resolution_clock::now(); // Inicia a medição de tempo.

    // Broadcast da matriz original para todos os processos.
    MPI_Bcast(matrix, N*N, MPI_INT, 0, MPI_COMM_WORLD);

    // Divide a matriz entre os processos.
    int rows_per_process = N / size; // Calcula o número de linhas que cada processo irá processar.
    int start_row = rank * rows_per_process; // Determina a linha inicial que cada processo irá processar.
    int end_row = (rank == size - 1) ? N : start_row + rows_per_process; // Determina a linha final (último processo pode pegar as linhas restantes).

    // Loop de múltiplas convoluções.
    for (int iter = 0; iter < ITERATIONS; iter++) {
        #pragma omp parallel for collapse(2) // Paraleliza a convolução usando OpenMP.
        for (int i = start_row; i < end_row; i++) {
            for (int j = 0; j < N; j++) {
                result[i][j] = apply_filter(i, j, matrix, filter); // Aplica o filtro na matriz local.
            }
        }
        // Sincroniza os processos MPI para garantir que a matriz esteja atualizada para a próxima iteração.
        MPI_Allgather(MPI_IN_PLACE, 0, MPI_DATATYPE_NULL, result[start_row], rows_per_process*N, MPI_INT, MPI_COMM_WORLD);

        // Copia a matriz resultante para a matriz original para a próxima iteração.
        #pragma omp parallel for collapse(2) // Paraleliza a cópia usando OpenMP.
        for (int i = start_row; i < end_row; i++) {
            for (int j = 0; j < N; j++) {
                matrix[i][j] = result[i][j];
            }
        }
    }

    // Sincroniza todos os processos antes de finalizar a medição de tempo.
    MPI_Barrier(MPI_COMM_WORLD);
    auto end = std::chrono::high_resolution_clock::now(); // Finaliza a medição de tempo.

    // Calcula o tempo de execução.
    std::chrono::duration<double> duration = end - start;
    double exec_time = duration.count(); // Tempo de execução em segundos.

    // Coleta os tempos de execução de todos os processos e exibe no processo mestre.
    double max_exec_time;
    MPI_Reduce(&exec_time, &max_exec_time, 1, MPI_DOUBLE, MPI_MAX, 0, MPI_COMM_WORLD);

    if (rank == 0) { // Apenas o processo mestre imprime o tempo de execução.
        std::cout << "A tarefa levou " << max_exec_time << " segundos para ser executado" << std::endl;
    }

    MPI_Finalize();  // Finaliza o ambiente MPI.
    return 0;  // Retorna 0, indicando que o programa terminou com sucesso.
}

```

Compile o programa dentro do Cluster:

```bash
mpic++ -fopenmp convolucao.cpp -o convolucao
```


### Criação de Scripts SLURM 

Crie um arquivo convolucao_limitado.slurm com a seguinte configuração:

```bash
#!/bin/bash
#SBATCH --job-name=convolucao_limitado    # Nome do job
#SBATCH --output=limitado_%j.txt   # Gera um novo arquivo de saída a cada execução
#SBATCH --ntasks=1                               # Número de tarefas (apenas 1 processos)
#SBATCH --cpus-per-task=1                        # Número de threads por tarefa (apenas 1 thread por processo)
#SBATCH --mem=512MB                             # Memória total alocada por nó (512 MB)
#SBATCH --time=00:10:00                          # Tempo máximo de execução (até 10 minutos)
#SBATCH --partition=espec                       # Fila do cluster a ser utilizada

# Exporta a variável de ambiente para configurar o número de threads OpenMP
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# Executa o programa usando MPI e OpenMP
mpirun ./convolucao
```
Este .slurm submete um job que executa 1 tarefas, com 1 CPU por tarefa, 512MB de memória por nó, e um tempo máximo de execução de 10 minutos.



Vamos solicitar mais recursos e ver o que acontece, crie um arquivo convolucao_buffado.slurm com a seguinte configuração:

```bash
#!/bin/bash
#SBATCH --job-name=convolucao_buffado             # Nome do job
#SBATCH --output=buffado_%j.txt          # Nome do arquivo de saída
#SBATCH --ntasks=4                              # Número de tarefas (4 processos)
#SBATCH --cpus-per-task=4                       # Número de threads por tarefa (2 threads por processo)
#SBATCH --mem=1024                               # Memória total alocada por nó (1024 MB = 1 GB)
#SBATCH --time=00:10:00                         # Tempo máximo de execução (10 minutos)
#SBATCH --partition=espec                      # fila do cluster a ser utilizada

# Exporta a variável de ambiente para configurar o número de threads OpenMP
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK


# Executa o programa usando MPI e OpenMP
mpirun ./convolucao
```
Este .slurm submete um job que executa 4 tarefas simultaneamente, com 4 CPUs por tarefa, 1 GB de memória por nó, e um tempo máximo de execução de 10 minutos.



### Exploração dos Comandos SLURM


Submeta os jobs usando o comando `sbatch`:

```bash
sbatch convolucao_limitado.slurm
sbatch convolucao_buffado.slurm
```

Execute o programa `convolucao` usando `srun`:

```bash
srun --ntasks=2 --cpus-per-task=2 --mem=500M ./convolucao
```
O comando srun está solicitando ao SLURM que aloque os recursos necessários para executar o programa convolucao diretamente via terminal. Ele solicita 2 tarefas para serem executadas, cada tarefa terá acesso a 2 CPUs (ou núcleos de processamento), e terá até 500 MB de memória disponível. 

Esse comando permite testar ou rodar programas interativamente com os recursos solicitados, podendo observar em tempo real o comportamento do programa.

Após iniciar o job com `srun`, use `sstat` para monitorar os recursos utilizados pelo job:

```bash
sstat --job <JOB_ID>
```

Utilize o comando `sinfo` para observar o estado dos nós e das filas:

```bash
sinfo
```

Use o comando `squeue` para observar o estado dos jobs em execução no Cluster Franky:

```bash
squeue
```

Use o comando `sprio` para visualizar a prioridade dos jobs:

```bash
sprio
```

Use `strigger` para definir um gatilho que notifique quando o job se aproximar do limite de tempo:

```bash
strigger --set --jobid=<JOB_ID> --threshold=60 --action="echo 'Job está próximo de atingir o limite de tempo!'"
```
**Explicando as flags**

--set: Indica que você está criando um novo gatilho. Este gatilho será ativado quando a condição especificada for atendida.

--jobid=<JOB_ID>: Especifica o ID do job para o qual o gatilho será configurado. 

--threshold=60: Define o limiar (threshold) de tempo restante para o job. Neste caso, o gatilho será acionado quando o job tiver 60 segundos restantes antes de atingir seu limite de tempo de execução. Sempre configure o tempo em segundos. 

--action="echo 'Job está próximo de atingir o limite de tempo!'": Especifica a ação que será executada quando o gatilho for ativado. Neste exemplo, a ação é um simples comando echo que imprimirá a mensagem "Job está próximo de atingir o limite de tempo!" no terminal ou no arquivo de saída do job.


Esse comando strigger define um gatilho que será acionado automaticamente quando o job especificado (<JOB_ID>) estiver a 60 segundos de atingir seu limite de tempo de execução. Quando o gatilho for ativado, ele executará o comando especificado na opção --action, que neste caso é imprimir uma mensagem de aviso. Se você tem um job que tem um tempo longo de execução e você não quer ficar monitorando, pode configurar esse gatilho para ser notificado quando o tempo do job estiver prestes a terminar.


!!! tip 
      Se quiser entender melhor sobre este comando e testar outros exemplos [verifique o material disponível aqui](../../Teoria/slurm.md)


### Atividade 02: Explorando comandos SLURM

1. Modifique o Número de Iterações da Convolução no Código convolucao.cpp; Aumente o número de iterações de    convolução no código para 30, 50, e 100. Isso aumentará a complexidade computacional, observe como o desempenho do código é afetado por essa modificação.

2. Ajuste o arquivo .slurm para garantir que os recursos de hardware (número de tarefas, CPUs por tarefa, e memória) e o tempo de execução sejam adequados para processar o aumento no número de iterações de convolução. Experimente diferentes configurações de recursos para observar como eles impactam o tempo de execução.

3. Gere um gráfico comparando o tempo de execução do algoritmo para cada número de iterações. Analise o impacto da alocação de mais recursos, como o número de CPUs e a quantidade de memória, no tempo de execução. 

4. Explore e explique como os comandos `sinfo`, `squeue`, `sprio`, `srun`, `sstat`, e `strigger` podem ser utilizados para monitorar e gerenciar jobs no cluster.

