# Explorando Comandos SLURM

**Objetivo:** Entender como a solicitação de recursos impacta o tempo de execução de uma tarefa e compreender como pedir recursos ao SLURM.

**Recursos Necessários:** Acesso ao cluster Franky configurado.

### Programa C++ para realizar a convolução de uma matriz

Crie um arquivo chamado `convolucao.cpp` com o seguinte conteúdo:

```cpp
#include <iostream>
#include <mpi.h>
#include <omp.h>
#include <chrono>
#include <cmath>

#define N 300
#define FILTER_SIZE 11
#define ITERATIONS 10

// Função para alocar dinamicamente uma matriz NxN
int** aloca_matriz(int n) {
    int** matriz = new int*[n];
    for (int i = 0; i < n; i++) {
        matriz[i] = new int[n]();
    }
    return matriz;
}

// Função para liberar memória da matriz NxN
void libera_matriz(int** matriz, int n) {
    if (matriz) {
        for (int i = 0; i < n; i++) {
            delete[] matriz[i];
        }
        delete[] matriz;
    }
}

// Função de convolução
int apply_filter(int x, int y, int** matrix, int filter[FILTER_SIZE][FILTER_SIZE]) {
    int result = 0;
    int filter_offset = FILTER_SIZE / 2;

    for (int i = -filter_offset; i <= filter_offset; i++) {
        for (int j = -filter_offset; j <= filter_offset; j++) {
            int xi = x + i;
            int yj = y + j;

            // Garantindo que os índices estão dentro dos limites da matriz
            if (xi >= 0 && xi < N && yj >= 0 && yj < N) {
                int value = matrix[xi][yj] * filter[i + filter_offset][j + filter_offset];
                result += value;
                result += std::sin(value) * std::cos(value); 
                result += static_cast<int>(std::pow(value, 2)) % 7;
            }
        }
    }
    return result;
}

int main(int argc, char *argv[]) {
    MPI_Init(&argc, &argv);

    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Aloca as matrizes dinamicamente
    int** matrix = aloca_matriz(N);
    int** result = aloca_matriz(N);
    int filter[FILTER_SIZE][FILTER_SIZE];

    // Inicializa a matriz apenas no processo mestre (rank 0)
    if (rank == 0) {
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                matrix[i][j] = (i + j) % 255; // Mantém valores entre 0-255
            }
        }

        for (int i = 0; i < FILTER_SIZE; i++) {
            for (int j = 0; j < FILTER_SIZE; j++) {
                filter[i][j] = 1; // Filtro simples
            }
        }
    }

    MPI_Barrier(MPI_COMM_WORLD);
    auto start = std::chrono::high_resolution_clock::now();

    // Enviar matriz para todos os processos (transmitindo linha por linha para evitar erros de buffer)
    for (int i = 0; i < N; i++) {
        MPI_Bcast(matrix[i], N, MPI_INT, 0, MPI_COMM_WORLD);
    }

    // Definir quantas linhas cada processo vai processar
    int rows_per_process = N / size;
    int start_row = rank * rows_per_process;
    int end_row = (rank == size - 1) ? N : start_row + rows_per_process;

    for (int iter = 0; iter < ITERATIONS; iter++) {
        #pragma omp parallel for collapse(2) schedule(dynamic, 1)
        for (int i = start_row; i < end_row; i++) {
            for (int j = 0; j < N; j++) {
                result[i][j] = apply_filter(i, j, matrix, filter);
            }
        }

        
        int recvcounts[size];  // Número de elementos que cada processo enviará
        int displs[size];      // Deslocamento de onde cada processo começa no buffer global

        for (int i = 0; i < size; i++) {
            recvcounts[i] = (i == size - 1) ? (N - i * rows_per_process) * N : rows_per_process * N;
            displs[i] = i * rows_per_process * N;
        }

        MPI_Gatherv(result[start_row], rows_per_process * N, MPI_INT,
                    matrix[0], recvcounts, displs, MPI_INT,
                    0, MPI_COMM_WORLD);
    }

    MPI_Barrier(MPI_COMM_WORLD);
    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double> duration = end - start;
    double exec_time = duration.count();

    double max_exec_time;
    MPI_Reduce(&exec_time, &max_exec_time, 1, MPI_DOUBLE, MPI_MAX, 0, MPI_COMM_WORLD);

    if (rank == 0) {
        std::cout << "A tarefa levou " << max_exec_time << " segundos para ser executada" << std::endl;
    }

    // Libera memória das matrizes
    libera_matriz(matrix, N);
    libera_matriz(result, N);

    MPI_Finalize();
    return 0;
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
#SBATCH --mem=2048                               # Memória total alocada por nó (2 GBs)
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

Utilize o comando `sinfo` para observar o estado dos nós e das filas:

```bash
sinfo
```

Use o comando `squeue` para observar o estado dos jobs em execução no Cluster Franky:

```bash
squeue
```

Use o comando abaixo para ver as especificações de recursos disponíveis nas filas do Cluster Franky:


```bash
scontrol show partition
```

Se quiser ver uma fila especifica, use o comando:

```bash
scontrol show partition nome_da_fila
```


###  Exemplo -> Interpretação da Partição `express`
Esta configuração define como os jobs são alocados e executados nos nós computacionais.

- **`PartitionName=express`** → Nome da partição.  
- **`AllowGroups=ALL`** → Todos os grupos de usuários podem submeter jobs.  
- **`AllowAccounts=ALL`** → Todos os usuários podem submeter jobs, independentemente da conta.  
- **`AllowQos=ALL`** → Todos os níveis de Qualidade de Serviço (**QoS**) são permitidos.  

- **`AllocNodes=ALL`** → Todos os nós podem ser alocados para jobs nesta partição.  
- **`Nodes=compute[00-04]`** → Os nós **compute00, compute01, compute02, compute03 e compute04** estão disponíveis.  

- **`Default=YES`** → Esta é a partição padrão se nenhuma for especificada.  
- **`MaxNodes=2`** → Cada job pode usar no máximo **2 nós**.  
- **`MinNodes=0`** → Jobs podem rodar mesmo se não houver nós alocados inicialmente.  
- **`MaxTime=00:05:00`** → Tempo máximo de execução para um job é **5 minutos**.  
- **`DefaultTime=NONE`** → O tempo padrão do job não está definido, então é obrigatório especificá-lo ao submeter.  

- **`MaxCPUsPerNode=UNLIMITED`** → Não há limite de CPUs por nó.  
- **`MaxCPUsPerSocket=UNLIMITED`** → Não há limite de CPUs por **socket** (conjunto de núcleos de um processador físico).  
- **`TotalCPUs=80`** → A partição possui **80 CPUs disponíveis no total**.  
- **`TotalNodes=5`** → Existem **5 nós disponíveis**.  
- **`DefMemPerNode=UNLIMITED`** → A memória disponível por nó não tem um limite padrão.  
- **`MaxMemPerNode=12000`** → Cada nó pode usar **até 12GB de RAM** por job.  
- **`TRES=cpu=80,mem=70000M,node=5,billing=80`** → O SLURM controla **80 CPUs, 70GB de RAM e 5 nós** nesta partição.  

- **`PriorityJobFactor=1`** → Todos os jobs na partição têm **fator de prioridade 1** (sem preferência sobre outras partições).  
- **`PriorityTier=1`** → Prioridade desta partição é **1** (normal).  

- **`ExclusiveUser=NO`** → Os nós podem ser compartilhados entre múltiplos jobs de diferentes usuários.  
- **`OverSubscribe=YES:4`** → Permite que até **4 jobs compartilhem um mesmo nó**.  
- **`OverTimeLimit=NONE`** → Não há limite para jobs que excedam o tempo solicitado.  
- **`PreemptMode=OFF`** → Jobs nesta partição **não podem ser interrompidos (preempted)** por outros jobs de maior prioridade.  

- **`DisableRootJobs=NO`** → Usuários root podem submeter jobs.  
- **`GraceTime=0`** → Sem tempo de espera antes da execução do job.  
- **`Hidden=NO`** → A partição está visível para todos os usuários.  
- **`RootOnly=NO`** → Não é necessário ser administrador para rodar jobs.  
- **`ReqResv=NO`** → Não requer reserva antecipada de recursos.  



Utilize o comando `scancel` para cancelar um job:

```bash
scancel jobid
```

Use o comando `sprio` para visualizar a prioridade dos jobs:

```bash
sprio
```


!!! tip 
      Se quiser testar outros exemplos [verifique o material disponível aqui](../../Teoria/slurm.md)


### Atividade 02: Explorando comandos SLURM

1. Modifique o Número de Iterações da Convolução no Código convolucao.cpp; Aumente o número de iterações de   convolução no código para 10, 30, e 60. Isso aumentará a complexidade computacional, observe como o desempenho do código é afetado por essa modificação.

2. Ajuste o arquivo .slurm para garantir que os recursos de hardware (número de tarefas, CPUs por tarefa, e memória) e o tempo de execução sejam adequados para processar o aumento no número de iterações de convolução. Experimente diferentes configurações de recursos para observar como eles impactam o tempo de execução.

3. Gere um gráfico comparando o tempo de execução do algoritmo para cada número de iterações. Analise o impacto da alocação de mais recursos, como o número de CPUs a quantidade de memória e observe também, como o volume de uso do cluster por outros usuários afeta o tempo de execução. 



# Entrega do relatório pelo Classroom ás 23h59 de 14/02