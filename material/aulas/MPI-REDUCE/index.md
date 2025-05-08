# Operações de Redução em MPI

Nesta aula, exploraremos o conceito de **redução distribuída de dados** em ambientes de computação paralela utilizando a biblioteca **MPI (Message Passing Interface)**. Nosso objetivo é compreender como distribuir tarefas entre múltiplos processos, coletar os resultados parciais e, por fim, consolidar esses resultados de forma eficiente.

Reduzir é um conceito clássico da programação funcional. A redução de dados envolve a redução de um conjunto de números em um conjunto menor de números por meio de uma função. Por exemplo, digamos que temos uma lista de números [1, 2, 3, 4, 5]. Reduzir esta lista de números com a função sum produziria sum([1, 2, 3, 4, 5]) = 15. Da mesma forma, a redução da multiplicação resultaria em multiplicar([1, 2, 3, 4, 5]) = 120.

Como você deve ter imaginado, pode ser muito complicado aplicar funções de redução em um conjunto de números distribuídos. Junto com isso, é difícil programar de forma eficiente reduções não comutativas, ou seja, reduções que devem ocorrer em uma ordem definida. Felizmente, o MPI tem uma função útil chamada MPI_Reduce que irá lidar com quase todas as reduções comuns que um programador precisa fazer em um aplicativo paralelo.



### Aproximação de PI com Comunicação Ponto-a-Ponto

Neste exemplo, cada processo MPI (exceto o coordenador) realiza uma simulação de Monte Carlo para estimar o valor de π. Cada processo gera coordenadas aleatórias (x, y) dentro do quadrado unitário e verifica se o ponto está dentro do quarto de círculo. O número de acertos é enviado ao processo coordenador por meio de `MPI_Send`, que os coleta com `MPI_Recv` e calcula π pela fórmula 𝜋 ≈ 4 \* (pontos dentro do círculo / total de pontos).

```cpp
#include <iostream>
#include <vector>
#include <cstdlib>
#include <cmath>
#include <mpi.h>

#define SEED_MPI 35791246

int main(int argc, char* argv[]) {
    const long niter = 10000000;
    int myid, nodenum, count = 0;
    double x, y, z, pi;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &myid);
    MPI_Comm_size(MPI_COMM_WORLD, &nodenum);

    std::vector<int> received(nodenum, 0);
    std::vector<long> recvniter(nodenum, 0);
    std::srand(SEED_MPI + myid);

    if (myid != 0) {
        for (long i = 0; i < niter; ++i) {
            x = static_cast<double>(std::rand()) / RAND_MAX;
            y = static_cast<double>(std::rand()) / RAND_MAX;
            z = std::sqrt(x * x + y * y);
            if (z <= 1.0) count++;
        }

        for (int i = 0; i < nodenum; ++i) {
            MPI_Send(&count, 1, MPI_INT, 0, 1, MPI_COMM_WORLD);
            MPI_Send(&niter, 1, MPI_LONG, 0, 2, MPI_COMM_WORLD);
        }
    } else {
        for (int i = 1; i < nodenum; ++i) {
            MPI_Recv(&received[i], 1, MPI_INT, i, 1, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
            MPI_Recv(&recvniter[i], 1, MPI_LONG, i, 2, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        }

        int finalcount = 0;
        long finalniter = 0;
        for (int i = 1; i < nodenum; ++i) {
            finalcount += received[i];
            finalniter += recvniter[i];
        }

        pi = static_cast<double>(finalcount) / finalniter * 4.0;
        std::cout << "Pi: " << pi << std::endl;
    }

    MPI_Finalize();
    return 0;
}

```
O programa `pi_send_receive.cpp` distribui o cálculo da aproximação de π entre múltiplos processos MPI. Cada processo-executor realiza um número fixo de iterações e envia os resultados para o processo rank 0 usando `MPI_Send`. O rank 0 coleta esses dados com `MPI_Recv` e realiza a agregação final.


Use `mpic++` para compilar seus programas:

```bash
mpic++ pi_send_receive.cpp -o pi_send_receive
```

Script de Submissão SLURM 

```bash
#!/bin/bash
#SBATCH --job-name=mpi_pi
#SBATCH --output=output_pi.txt
#SBATCH --ntasks=16
#SBATCH --time=00:05:00
#SBATCH --partition=normal

mpirun -np $SLURM_NTASKS ./pi_send_receive
```

Submeta com:

```bash
sbatch job_pi.sh
```



### Agregação com MPI\_Reduce

Neste segundo exemplo, cada processo gera números aleatórios e calcula a soma local. Em vez de enviar esses valores individualmente ao rank 0, todos os processos participam de uma operação de redução coletiva com `MPI_Reduce`. A operação soma (`MPI_SUM`) os valores locais e entrega o total apenas ao rank 0, que então calcula a média global.

```cpp
#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <cassert>
#include <mpi.h>

std::vector<float> create_rand_nums(int num_elements) {
    std::vector<float> rand_nums(num_elements);
    for (int i = 0; i < num_elements; ++i) {
        rand_nums[i] = static_cast<float>(std::rand()) / RAND_MAX;
    }
    return rand_nums;
}

int main(int argc, char** argv) {
    if (argc != 2) {
        std::cerr << "Usage: avg num_elements_per_proc" << std::endl;
        return 1;
    }

    int num_elements_per_proc = std::atoi(argv[1]);

    MPI_Init(nullptr, nullptr);
    int world_rank, world_size;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    std::srand(static_cast<unsigned int>(std::time(nullptr)) * (world_rank + 1));
    std::vector<float> rand_nums = create_rand_nums(num_elements_per_proc);

    float local_sum = 0.0f;
    for (float val : rand_nums) {
        local_sum += val;
    }

    std::cout << "Local sum for process " << world_rank
              << " = " << local_sum
              << ", avg = " << (local_sum / num_elements_per_proc) << std::endl;

    float global_sum = 0.0f;
    MPI_Reduce(&local_sum, &global_sum, 1, MPI_FLOAT, MPI_SUM, 0, MPI_COMM_WORLD);

    if (world_rank == 0) {
        float global_avg = global_sum / (world_size * num_elements_per_proc);
        std::cout << "Total sum = " << global_sum
                  << ", global avg = " << global_avg << std::endl;
    }

    MPI_Barrier(MPI_COMM_WORLD);
    MPI_Finalize();
    return 0;
}

```

#### O que são primitivas coletivas?

Primitivas coletivas são funções da biblioteca MPI que envolvem todos os processos de um grupo (geralmente `MPI_COMM_WORLD`) de forma coordenada. Ao contrário das comunicações ponto-a-ponto, em que processos individuais trocam mensagens diretamente entre si (`MPI_Send` e `MPI_Recv`), as primitivas coletivas realizam operações em grupo — como distribuição de dados, coleta de resultados ou sincronização.

Entre as principais primitivas coletivas, temos:

* `MPI_Bcast`: envio de dados de um processo para todos os outros
* `MPI_Gather` / `MPI_Scatter`: coleta ou distribuição de dados entre processos
* `MPI_Reduce` / `MPI_Allreduce`: agregação (redução) de valores de todos os processos
* `MPI_Barrier`: sincronização de todos os processos

Essas funções são otimizadas internamente para oferecer alto desempenho em ambientes paralelos e facilitam a implementação de algoritmos distribuídos.

A função `MPI_Reduce` permite que cada processo envie seus dados diretamente para uma operação de redução (como soma, mínimo, máximo) que é executada automaticamente no processo raiz. Isso elimina a necessidade de `MPI_Send` e `MPI_Recv` manuais, tornando o código mais limpo e eficiente.

Este proximo exemplo estende o uso das primitivas coletivas para um cenário em que **todos os processos** precisam conhecer o resultado global. Utilizamos `MPI_Allreduce` para calcular a média global e, em seguida, `MPI_Reduce` para somar os desvios quadráticos ao redor da média. O desvio padrão é calculado no rank0 a partir desses valores.

```cpp
#include <iostream>
#include <vector>
#include <cstdlib>
#include <cmath>
#include <ctime>
#include <cassert>
#include <mpi.h>

std::vector<float> create_rand_nums(int num_elements) {
    std::vector<float> rand_nums(num_elements);
    for (int i = 0; i < num_elements; ++i) {
        rand_nums[i] = static_cast<float>(std::rand()) / RAND_MAX;
    }
    return rand_nums;
}

int main(int argc, char** argv) {
    if (argc != 2) {
        std::cerr << "Usage: avg num_elements_per_proc" << std::endl;
        return 1;
    }

    int num_elements_per_proc = std::atoi(argv[1]);

    MPI_Init(nullptr, nullptr);
    int world_rank, world_size;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    std::srand(static_cast<unsigned int>(std::time(nullptr)) * (world_rank + 1));
    std::vector<float> rand_nums = create_rand_nums(num_elements_per_proc);

    float local_sum = 0.0f;
    for (float num : rand_nums)
        local_sum += num;

    float global_sum = 0.0f;
    MPI_Allreduce(&local_sum, &global_sum, 1, MPI_FLOAT, MPI_SUM, MPI_COMM_WORLD);
    float mean = global_sum / (num_elements_per_proc * world_size);

    float local_sq_diff = 0.0f;
    for (float num : rand_nums)
        local_sq_diff += (num - mean) * (num - mean);

    float global_sq_diff = 0.0f;
    MPI_Reduce(&local_sq_diff, &global_sq_diff, 1, MPI_FLOAT, MPI_SUM, 0, MPI_COMM_WORLD);

    if (world_rank == 0) {
        float stddev = std::sqrt(global_sq_diff / (num_elements_per_proc * world_size));
        std::cout << "Mean = " << mean << ", Standard deviation = " << stddev << std::endl;
    }

    MPI_Barrier(MPI_COMM_WORLD);
    MPI_Finalize();
    return 0;
}

```

Enquanto `MPI_Reduce` entrega o resultado da operação ao processo rank0, `MPI_Allreduce` entrega o mesmo resultado a todos os processos. Isso é útil quando todos os nós precisam do valor reduzido, como ao calcular desvios padrão, médias globais e sincronizações de estado.

O código para cálculo do desvio padrão utiliza duas reduções: uma com `MPI_Allreduce` para obter a média global e outra com `MPI_Reduce` para calcular a soma dos desvios quadráticos.



### Sua vez!

1-  Refatore o código do primeiro exemplo para utilizar `MPI_Reduce` ao invés de `MPI_Send` e `MPI_Recv`.

2- Faça um código que utilize MPI e OpenMP em que:
    * O processo Rank0 calcula o número de pontos com 16 threads OpenMP.
    * Os resultados dos outros processos são coletados via `MPI_Reduce`.



**Faça a sua subimissão até as 23h59 de 19/05** 

