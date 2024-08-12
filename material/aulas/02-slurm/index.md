# Aula 02: Explorando Comandos SLURM

**Objetivo:** Entender como a solicitação recursos impacta o tempo de execução de uma tarefa e explorar a funcionalidade dos principais comandos SLURM.

**Recursos Necessários:** Acesso ao cluster Franky configurado.

### Parte 1: Programa C++ para Percorrer uma Árvore de Decisão

Crie um arquivo chamado `decision_tree.cpp` com o seguinte conteúdo:

```cpp
#include <iostream>   // Inclui a biblioteca de entrada e saída padrão (para usar std::cout).
#include <vector>     // Inclui a biblioteca de vetores (não usada diretamente neste código, mas útil para estruturas de dados).
#include <chrono>     // Inclui a biblioteca para medição de tempo (usada para cronometrar a execução do código).
#include <thread>     // Inclui a biblioteca para trabalhar com threads e simular atrasos (usada para criar uma pausa no processamento).
#include <mpi.h>      // Inclui a biblioteca MPI para paralelismo entre múltiplos nós.
#include <omp.h>      // Inclui a biblioteca OpenMP para paralelismo com threads.

// Define uma estrutura chamada Node que representa um nó em uma árvore binária.
struct Node {
    int value;       // Valor armazenado no nó.
    Node* left;      // Ponteiro para o nó filho à esquerda.
    Node* right;     // Ponteiro para o nó filho à direita.
    
    // Construtor do nó, inicializa o valor e define os filhos como nulos.
    Node(int val) : value(val), left(nullptr), right(nullptr) {}
};

// Função recursiva para criar uma árvore binária de determinada profundidade.
Node* createTree(int depth) {
    if (depth == 0) return nullptr;  // Se a profundidade for 0, retorna nullptr (árvore vazia).
    
    Node* node = new Node(depth);    // Cria um novo nó com o valor igual à profundidade atual.
    
    // Cria recursivamente a subárvore esquerda e direita, diminuindo a profundidade em 1.
    node->left = createTree(depth - 1);
    node->right = createTree(depth - 1);
    
    return node;  // Retorna o nó criado.
}

// Função recursiva para percorrer a árvore binária (traversal) com print do nível atual.
void traverseTree(Node* root, int level) {
    if (root == nullptr) return;  // Se o nó for nulo, a função retorna (caso base da recursão).
    
    // Imprime o nível atual da árvore.
    std::cout << "Nível da árvore: " << level << " - Processado por thread " << omp_get_thread_num() << std::endl;
    
    // Simula um pequeno atraso para representar o processamento do nó atual.
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    
    // Paraleliza a travessia das subárvores esquerda e direita usando OpenMP.
    #pragma omp parallel sections
    {
        #pragma omp section
        {
            traverseTree(root->left, level + 1);  // Percorre recursivamente a subárvore esquerda, aumentando o nível.
        }
        #pragma omp section
        {
            traverseTree(root->right, level + 1); // Percorre recursivamente a subárvore direita, aumentando o nível.
        }
    }
}

// Função principal do programa.
int main(int argc, char* argv[]) {
    MPI_Init(&argc, &argv);  // Inicializa o ambiente MPI.

    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);  // Obtém o número total de processos.
    
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);  // Obtém o identificador do processo atual.

    int depth = 20; // Define a profundidade da árvore como 20.

    Node* root = nullptr;

    // Apenas o processo mestre (rank 0) cria a árvore.
    if (world_rank == 0) {
        root = createTree(depth);
    }

    // Marca o tempo inicial antes de começar a percorrer a árvore.
    auto start = std::chrono::high_resolution_clock::now();
    
    // Distribui o trabalho para percorrer a árvore entre os processos usando MPI.
    if (world_rank == 0) {
        // Processo mestre percorre a subárvore esquerda e envia a subárvore direita para outro processo.
        int right_process = 1 % world_size;  // Atribui a subárvore direita ao próximo processo.
        MPI_Send(root->right, sizeof(Node), MPI_BYTE, right_process, 0, MPI_COMM_WORLD);
        traverseTree(root->left, 0);
    } else {
        // Outros processos recebem uma parte da árvore e a processam.
        Node* received_node = new Node(0);
        MPI_Recv(received_node, sizeof(Node), MPI_BYTE, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        traverseTree(received_node, 0);
    }

    // Marca o tempo final após concluir a travessia da árvore.
    auto end = std::chrono::high_resolution_clock::now();
    
    // Apenas o processo mestre calcula e exibe o tempo de execução.
    if (world_rank == 0) {
        std::chrono::duration<double> duration = end - start;
        std::cout << "Tempo de execução: " << duration.count() << " segundos" << std::endl;
    }

    MPI_Finalize();  // Finaliza o ambiente MPI.
    return 0;  // Retorna 0, indicando que o programa terminou com sucesso.
}

```

Compile o programa dentro do Cluster:

```bash
mpic++ -fopenmp decision_tree.cpp -o decision_tree
```


### Criação de Scripts SLURM 

Crie um arquivo tree_limitado.slurm com a seguinte configuração:

```bash
#!/bin/bash
#SBATCH --job-name=tree_limitado    # Nome do job
#SBATCH --output=tree_limitado_output_%j.txt  # Nome do arquivo de saída
#SBATCH --ntasks=2                               # Número de tarefas (apenas 2 processos)
#SBATCH --cpus-per-task=1                        # Número de threads por tarefa (apenas 1 thread por processo)
#SBATCH --mem=1024                               # Memória total alocada por nó (1024 MB = 1 GB)
#SBATCH --time=00:30:00                          # Tempo máximo de execução (até meia hora)
#SBATCH --partition=espec                       # Fila do cluster a ser utilizada

# Exporta a variável de ambiente para configurar o número de threads OpenMP
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# Executa o programa usando MPI e OpenMP
mpirun ./decision_tree
```
Este .slurm submete um job que executa 2 tarefas simultaneamente, com 1 CPU por tarefa, 1024 MB (1 GB) de memória por nó, e um tempo máximo de execução de 30 minutos.



Vamos solicitar mais recursos e ver o que acontece, crie um arquivo tree_buffado.slurm com a seguinte configuração:

```bash
#!/bin/bash
#SBATCH --job-name=tree_buffado       # Nome do job
#SBATCH --output=ttree_buffado_%j.txt  # Nome do arquivo de saída
#SBATCH --ntasks=4                              # Número de tarefas (4 processos)
#SBATCH --cpus-per-task=2                       # Número de threads por tarefa (2 threads por processo)
#SBATCH --mem=2048                               # Memória total alocada por nó (2048 MB = 2 GB)
#SBATCH --time=00:30:00                         # Tempo máximo de execução (30 minutos)
#SBATCH --partition=espec                      # fila do cluster a ser utilizada

# Exporta a variável de ambiente para configurar o número de threads OpenMP
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK


# Executa o programa usando MPI e OpenMP
mpirun ./decision_tree
```
Este .slurm submete um job que executa tarefas simultaneamente, com 2 CPUs por tarefa, 2 GB de memória por nó, e um tempo máximo de execução de 30 minutos.



### Exploração dos Comandos SLURM


Submeta os jobs usando o comando `sbatch`:

```bash
sbatch tree_limitado.slurm
sbatch tree_buffado.slurm
```

Execute o programa `decision_tree` usando `srun`:

```bash
srun --ntasks=1 --cpus-per-task=2 --mem=500M ./decision_tree
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

1. Modifique a Profundidade da Árvore no código `decision_tree.cpp` para 30, 40 e 50.

2. Edite o arquivo `.slurm` para garantir que os recursos de hardware (CPUs, memória) e o tempo de execução sejam adequados para as diferentes profundidades da árvore.

3. Gere um gráfico comparando o tempo de execução do algoritmo para cada profundidade da árvore. Analise o impacto da alocação de mais recursos, como CPUs e memória, no tempo de execução.

4. Explore e explique como os comandos `sinfo`, `squeue`, `sprio`, `srun`, `sstat`, e `strigger` podem ser utilizados para monitorar e gerenciar jobs em um cluster.
