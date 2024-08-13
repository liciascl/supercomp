# Aula 02: Explorando Comandos SLURM

**Objetivo:** Entender como a solicitação de recursos impacta o tempo de execução de uma tarefa e compreender como pedir recursos ao SLURM.

**Recursos Necessários:** Acesso ao cluster Franky configurado.

### Programa C++ para somar os valores de uma matriz

Crie um arquivo chamado `soma_matriz.cpp` com o seguinte conteúdo:

```cpp
#include <iostream>   // Inclui a biblioteca padrão de entrada e saída.
#include <mpi.h>      // Inclui a biblioteca MPI para paralelismo entre múltiplos processos.
#include <omp.h>      // Inclui a biblioteca OpenMP para paralelismo com threads.
#include <chrono>     // Inclui a biblioteca para medição de tempo.

// Função principal do programa.
int main(int argc, char *argv[]) {
    MPI_Init(&argc, &argv);  // Inicializa o ambiente MPI.

    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);  // Obtém o rank (identificador) do processo atual.
    MPI_Comm_size(MPI_COMM_WORLD, &size);  // Obtém o número total de processos.

    const int N = 200;  // Define a dimensão da matriz NxN.
    int data[N][N];    // Declara a matriz de dados a ser processada.

    // Inicializa a medição de tempo
    auto start = std::chrono::high_resolution_clock::now();

    // Inicialização da matriz pelo processo de rank 0 (processo mestre).
    if (rank == 0) {
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                data[i][j] = i + j;  // Inicializa cada elemento com a soma dos índices.
            }
        }

        // Imprime a matriz inicial no processo 0.
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                std::cout << data[i][j] << " ";
            }
            std::cout << std::endl;
        }
        std::cout << std::endl << std::endl;
    }

    // Divide a matriz entre os processos.
    int chunk_size = N / size;  // Calcula o número de linhas que cada processo receberá.
    int local_data[chunk_size][N];  // Declara a matriz local para cada processo.

    // Distribui partes iguais da matriz original para todos os processos.
    MPI_Scatter(&data, chunk_size * N, MPI_INT, &local_data, chunk_size * N, MPI_INT, 0, MPI_COMM_WORLD);

    // Paraleliza o processamento da matriz local com OpenMP.
    #pragma omp parallel for collapse(2)
    for (int i = 0; i < chunk_size; i++) {
        for (int j = 0; j < N; j++) {
            local_data[i][j] *= local_data[i][j];  // Eleva ao quadrado cada elemento da matriz local.
        }
    }

    // Reúne os dados processados de todos os processos no processo 0.
    MPI_Gather(&local_data, chunk_size * N, MPI_INT, &data, chunk_size * N, MPI_INT, 0, MPI_COMM_WORLD);

    // O processo 0 imprime a matriz final processada e o tempo de execução.
    if (rank == 0) {
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                std::cout << data[i][j] << " ";
            }
            std::cout << std::endl;
        }

        // Calcula e imprime o tempo de execução
        auto end = std::chrono::high_resolution_clock::now();
        std::chrono::duration<double> duration = end - start;
        std::cout << "Tempo de execução: " << duration.count() << " segundos" << std::endl;
    }

    MPI_Finalize();  // Finaliza o ambiente MPI.
    return 0;  // Retorna 0, indicando que o programa terminou com sucesso.
}

```

Compile o programa dentro do Cluster:

```bash
mpic++ -fopenmp soma_matriz.cpp -o soma_matriz
```


### Criação de Scripts SLURM 

Crie um arquivo soma_matriz_limitado.slurm com a seguinte configuração:

```bash
#!/bin/bash
#SBATCH --job-name=matriz_limitado    # Nome do job
#SBATCH --output=matriz_limitado_output_%j.txt   # Gera um novo arquivo de saída a cada execução
#SBATCH --ntasks=2                               # Número de tarefas (apenas 2 processos)
#SBATCH --cpus-per-task=1                        # Número de threads por tarefa (apenas 1 thread por processo)
#SBATCH --mem=512MB                             # Memória total alocada por nó (512 MB)
#SBATCH --time=00:10:00                          # Tempo máximo de execução (até 10 minutos)
#SBATCH --partition=espec                       # Fila do cluster a ser utilizada

# Exporta a variável de ambiente para configurar o número de threads OpenMP
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# Executa o programa usando MPI e OpenMP
mpirun ./soma_matriz
```
Este .slurm submete um job que executa 2 tarefas simultaneamente, com 1 CPU por tarefa, 512MB de memória por nó, e um tempo máximo de execução de 10 minutos.



Vamos solicitar mais recursos e ver o que acontece, crie um arquivo matriz_buffado.slurm com a seguinte configuração:

```bash
#!/bin/bash
#SBATCH --job-name=matriz_buffado             # Nome do job
#SBATCH --output=ttree_buffado_%j.txt          # Nome do arquivo de saída
#SBATCH --ntasks=4                              # Número de tarefas (4 processos)
#SBATCH --cpus-per-task=2                       # Número de threads por tarefa (2 threads por processo)
#SBATCH --mem=1024                               # Memória total alocada por nó (1024 MB = 1 GB)
#SBATCH --time=00:10:00                         # Tempo máximo de execução (10 minutos)
#SBATCH --partition=espec                      # fila do cluster a ser utilizada

# Exporta a variável de ambiente para configurar o número de threads OpenMP
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK


# Executa o programa usando MPI e OpenMP
mpirun ./soma_matriz
```
Este .slurm submete um job que executa 4 tarefas simultaneamente, com 2 CPUs por tarefa, 1 GB de memória por nó, e um tempo máximo de execução de 10 minutos.



### Exploração dos Comandos SLURM


Submeta os jobs usando o comando `sbatch`:

```bash
sbatch matriz_limitado.slurm
sbatch matriz_buffado.slurm
```

Execute o programa `soma_matriz` usando `srun`:

```bash
srun --ntasks=1 --cpus-per-task=2 --mem=500M ./soma_matriz
```
O comando srun está solicitando ao SLURM que aloque os recursos necessários para executar o programa decision_tree diretamente via terminal. Ele solicita 1 tarefa para ser executada, a tarefa terá acesso a 2 CPUs (ou núcleos de processamento), terá até 500 MB de memória disponível. 

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

1. Modifique a dimensão da matriz no código `soma_matriz.cpp` para 300x300, 450x450 e 500x500.

2. Modifique o arquivo .slurm para garantir que os recursos de hardware (número de tarefas MPI, CPUs por tarefa, e memória) e o tempo de execução sejam adequados para processar as diferentes dimensões da matriz.

3. Gere um gráfico comparando o tempo de execução do algoritmo para cada dimensão da matriz. Analise o impacto da alocação de mais recursos, como o número de CPUs e a quantidade de memória, no tempo de execução.

4. Explore e explique como os comandos `sinfo`, `squeue`, `sprio`, `srun`, `sstat`, e `strigger` podem ser utilizados para monitorar e gerenciar jobs em um cluster.

