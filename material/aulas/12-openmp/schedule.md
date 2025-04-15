
# Trabalhando com Schedulers no OpenMP
No OpenMP, schedulers controlam a distribuição de iterações de loops entre threads. A primeira tarefa será entender como diferentes schedulers funcionam.


Tenha o código abaixo disponível na sua pasta scratch no Cluster Franky

omp_schedulers.cpp

```cpp

#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <omp.h>
void print_iterations(const std::string& description,
const std::vector< std::vector<int> >& vectors,
const int n)
{
    std::vector< std::string > strings(4, std::string());
for (int i = 0; i != n; i++)
    {
for (int j = 0; j != 4; j++)
        {
const auto& vector = vectors[j];
auto it = std::find(vector.begin(), vector.end(), i);
if (it != vector.end())
            {
                strings[j] += "*";
            }
else 
            { 
                strings[j] += " ";
            }
        }
    }
    std::cout << description << std::endl;
for (auto& s : strings)
    {
        std::cout << s << "\n";
    }
    std::cout << std::endl;
}
template <typename T>
void schedule(T function, 
const std::string& description, 
const int n)
{
    std::vector< std::vector<int> > vectors(4, std::vector<int>());
function(vectors, n);
print_iterations(description, vectors, n);
}
void scheduleDefault(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for 
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleStatic(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
        #pragma omp for schedule(static)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleStatic4(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(static, 4)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleStatic8(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(static, 8)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleDynamic(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(dynamic)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleDynamic1(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(dynamic, 1)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleDynamic4(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(dynamic, 4)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleDynamic8(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(dynamic, 8)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleGuided(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(guided)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleGuided2(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(guided, 2)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleGuided4(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(guided, 4)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleGuided8(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(guided, 8)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleAuto(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(auto)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
void scheduleRuntime(std::vector< std::vector<int> >& vectors,
int n)
{
#pragma omp parallel num_threads(4), shared(vectors, n)
    {    
#pragma omp for schedule(auto)
for (int i = 0; i < n; i++)
        {
            vectors[omp_get_thread_num()].push_back(i);
        }
    }
}
int main()
{
const int n = 64;
schedule(scheduleDefault,  "default:               ", n);
schedule(scheduleStatic,   "schedule(static):      ", n);
schedule(scheduleStatic4,  "schedule(static, 4):   ", n);
schedule(scheduleStatic8,  "schedule(static, 8):   ", n);
schedule(scheduleDynamic,  "schedule(dynamic):     ", n);
schedule(scheduleDynamic1, "schedule(dynamic, 1):  ", n);
schedule(scheduleDynamic4, "schedule(dynamic, 4):  ", n);
schedule(scheduleDynamic8, "schedule(dynamic, 8):  ", n);
schedule(scheduleGuided,   "schedule(guided):      ", n);
schedule(scheduleGuided2,  "schedule(guided, 2):   ", n);
schedule(scheduleGuided4,  "schedule(guided, 4):   ", n);
schedule(scheduleGuided8,  "schedule(guided, 8):   ", n);
schedule(scheduleAuto,     "schedule(auto):        ", n);
schedule(scheduleRuntime,  "schedule(runtime):     ", n);
return 0;
}

```



Compile o código no cluster com suporte ao OpenMP:

```bash
g++ -fopenmp omp_schedulers.cpp -o omp_schedulers
```

Crie um arquivo de submissão de job (`submit_job.sh`) com o seguinte conteúdo:

   ```bash
   #!/bin/bash
   #SBATCH --job-name=omp_scheduler_test
   #SBATCH --output=output_omp_schedulers.txt
   #SBATCH --ntasks=1
   #SBATCH --cpus-per-task=4
   #SBATCH --time=00:05:00

   ./omp_schedulers
   ```

Submeta o job ao Slurm:

```bash
sbatch submit_job.sh
```

Verifique os resultados no arquivo `output_omp_schedulers.txt`. Compare o output obtido com o exemplo fornecido, analisando como cada tipo de `scheduler` distribui as iterações dos loops.

### Analisando os Schedulers no OpenMP

Cada scheduler do OpenMP se comporta de maneira diferente, e você deve observar o impacto de cada um:

   **static**: As iterações são divididas igualmente entre as threads.
   
   **dynamic**: As threads pegam blocos de iterações conforme terminam o trabalho.
   
   **guided**: Distribui blocos maiores no início e menores no final, equilibrando a carga.
   
   **auto**: Deixa o compilador escolher a melhor estratégia.
   
   **runtime**: Usa a estratégia definida em tempo de execução.

### Entrega da Atividade 

### Schedules

1. Execute o código com diferentes schedulers (static, dynamic, guided, auto) e registre o tempo necessário para cada execução, faça ao menos 3 execuções para cada teste.

2. Compare os tempos médios de execução para cada scheduler.
  
**Perguntas de Análise**:

- Qual scheduler apresentou o menor tempo médio?

- Algum scheduler teve variações significativas entre as execuções? Se sim, por quê?

- Alguma característica específica do trabalho (como carga de dados, balanceamento) parece ter influenciado o comportamento de um scheduler em particular?


**Faça um relatório com as suas análises e entregue até as 23h59 de 24/04** 