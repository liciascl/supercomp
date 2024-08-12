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
    std::cout << "Nível da árvore: " << level << std::endl;
    
    // Simula um pequeno atraso para representar o processamento do nó atual.
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    
    traverseTree(root->left, level + 1);  // Percorre recursivamente a subárvore esquerda, aumentando o nível.
    traverseTree(root->right, level + 1); // Percorre recursivamente a subárvore direita, aumentando o nível.
}

// Função principal do programa.
int main() {
    int depth = 20; // Define a profundidade da árvore como 20.
    
    // Cria uma árvore binária com a profundidade especificada.
    Node* root = createTree(depth);
    
    // Marca o tempo inicial antes de começar a percorrer a árvore.
    auto start = std::chrono::high_resolution_clock::now();
    
    // Percorre toda a árvore binária, começando do nível 0.
    traverseTree(root, 0);
    
    // Marca o tempo final após concluir a travessia da árvore.
    auto end = std::chrono::high_resolution_clock::now();
    
    // Calcula a duração total da travessia da árvore.
    std::chrono::duration<double> duration = end - start;
    
    // Imprime o tempo de execução em segundos.
    std::cout << "Tempo de execução: " << duration.count() << " segundos" << std::endl;
    
    return 0;  // Retorna 0, indicando que o programa terminou com sucesso.
}
```

Compile o programa dentro do Cluster:

```bash
g++ decision_tree.cpp -o decision_tree
```


### Criação de Scripts SLURM com Diferentes Recursos

Crie um arquivo chamado `low_resources.slurm` com o seguinte conteúdo:

```bash
#!/bin/bash
#SBATCH --job-name=low_resources_job          # Define o nome do job como 'low_resources_job'.
#SBATCH --output=low_resources_output_%j.txt  # Especifica o arquivo de saída para o job, incluindo o ID do job (%j).
#SBATCH --ntasks=1                            # Define o número de tarefas para o job como 1.
#SBATCH --cpus-per-task=1                     # Aloca 1 CPU por tarefa.
#SBATCH --mem=300M                            # Solicita 300 MB de memória.
#SBATCH --time=00:10:00                       # Define o tempo máximo de execução do job como 10 minutos.

./decision_tree                               # Executa o programa 'decision_tree'.
```
Este .slurm submete um job que executa o programa decision_tree com 1 CPU, 300MB de memória, 10 minutos de tempo de execução.

Vamos criar um arquivo solicitando mais recursos para ver o que acontece; Crie um arquivo chamado `high_resources.slurm` com o seguinte conteúdo:

```bash
#!/bin/bash
#SBATCH --job-name=high_resources_job          # Define o nome do job como 'high_resources_job'.
#SBATCH --output=high_resources_output_%j.txt  # Especifica o arquivo de saída para o job, incluindo o ID do job (%j).
#SBATCH --ntasks=2                             # Define o que cada job execute 2 tarefas.
#SBATCH --cpus-per-task=2                      # Aloca 2 CPUs por tarefa.
#SBATCH --mem=1000M                            # Solicita 1000 MB (1 GB) de memória.
#SBATCH --time=00:10:00                        # Define o tempo máximo de execução do job como 10 minutos.

./decision_tree                                # Executa o programa 'decision_tree'.
```
Este .slurm submete um job que executa 2 tarefas ao mesmo tempo, com 2 CPU, 1GB de memória, 10 minutos de tempo de execução.

### Exploração dos Comandos SLURM


Submeta os jobs usando o comando `sbatch`:

```bash
sbatch low_resources.slurm
sbatch high_resources.slurm
```

Execute o programa `decision_tree` usando `srun`:

```bash
srun --ntasks=1 --cpus-per-task=2 --mem=500M ./decision_tree
```
O comando srun está solicitando ao SLURM que aloque os recursos necessários para executar o programa decision_tree diretamente via terminal.

Especificamente:

    1. Ele solicita 1 tarefa para ser executada.
    2. A tarefa terá acesso a 2 CPUs (ou núcleos de processamento).
    3. A tarefa terá até 500 MB de memória disponível.
    4. Assim que os recursos estiverem disponíveis, o programa decision_tree será executado.

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
