
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

### Paralelizando Códigos Recursivos

Vamos agora focar na paralelização de código recursivo, como o cálculo do Pi.

Examine o código `pi_recursivo.cpp` e identifique oportunidades de paralelismo:

```cpp
#include <omp.h>
#include <iostream>
#include <iomanip>
static long num_steps = 1024l*1024*1024*2;

#define MIN_BLK  1024*1024*256

double sum = 0;

void pi_r(long Nstart, long Nfinish, double step) {
    long i,iblk;
    if (Nfinish-Nstart < MIN_BLK){
        for (i = Nstart; i < Nfinish; i++){
            double x = (i+0.5)*step;
            sum += 4.0/(1.0+x*x); 
        }
    } else {
        iblk = Nfinish-Nstart;
        pi_r(Nstart,         Nfinish-iblk/2,step);
        pi_r(Nfinish-iblk/2, Nfinish,       step);
    }
}

int main () {
    long i;
    double step, pi;
    double init_time, final_time;
    step = 1.0/(double) num_steps;
    init_time = omp_get_wtime();
    pi_r(0, num_steps, step);
    pi = step * sum;
    final_time = omp_get_wtime() - init_time;

    std::cout << "for " << num_steps << " steps pi = " << std::setprecision(15) << pi << " in " << final_time << " secs\n";
}
```

**Para refletir**:

   1. Existem dependências que dificultam a paralelização?

   2. Quantas chamadas recursivas são feitas?

   3. Onde é possível inserir paralelismo no código?



### Paralelizando com `for parallel`

Analise onde o loop pode ser paralelizado e adicione a diretiva `#pragma omp parallel for`. Meça o tempo de execução com diferentes valores de `MIN_BLK` e veja o impacto no desempenho.

   **Dica**: O valor de `MIN_BLK` pode influenciar significativamente o desempenho, já que um valor muito baixo aumenta a sobrecarga de criação de tarefas, enquanto um valor muito alto reduz o paralelismo.

### Paralelizando com `task`

Adicione a diretiva `#pragma omp task` para paralelizar as chamadas recursivas no código. Utilize `#pragma omp taskwait` para garantir que todas as tarefas sejam concluídas antes de seguir em frente.

   **Experimente diferentes números de tarefas** e compare os ganhos de desempenho com a versão paralela utilizando `for`.


### Lidando com Efeitos Colaterais

Programas paralelos podem ter efeitos colaterais, como no caso de modificações em estruturas compartilhadas entre threads (por exemplo, o vetor `vec` no código `vetor_insert.cpp`).

```cpp
#include <vector>
#include <iostream>
#include <unistd.h>


double conta_complexa(int i) {
	return 2 * i;
}

int main() {
	int N = 10000; 
	std::vector<double> vec;
	for (int i = 0; i < N; i++) {
		vec.push_back(conta_complexa(i));
	}
	
	for (int i = 0; i < N; i++) {
		std::cout << i << " ";
	}
	
	return 0;
}
```

Para evitar conflitos, podemos usar regiões críticas:

```cpp
#pragma omp critical
{
    vec.push_back(conta_complexa(i));
}
```

Mas cuidado! Incluir código demais dentro de uma seção crítica pode anular os ganhos de paralelismo. Procure isolar apenas a parte que realmente necessita de exclusividade.

### Pré-Alocação de Memória

Uma estratégia mais eficiente é alocar memória previamente, evitando o uso de `push_back` em regiões críticas. Isso pode ser feito da seguinte maneira:

```cpp
vec.resize(N);
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    vec[i] = conta_complexa(i);
}
```

Com isso, você elimina a necessidade de sincronização entre threads, melhorando o desempenho.

### Entrega da Atividade 

### Schedules

1. Execute o código com diferentes schedulers (static, dynamic, guided, auto) e registre o tempo necessário para cada execução, faça ao menos 3 execuções para cada teste.

2. Compare os tempos médios de execução para cada scheduler.
  
**Perguntas de Análise**:

- Qual scheduler apresentou o menor tempo médio?

- Algum scheduler teve variações significativas entre as execuções? Se sim, por quê?

- Alguma característica específica do trabalho (como carga de dados, balanceamento) parece ter influenciado o comportamento de um scheduler em particular?

### Calculo do PI

1. Paralelize o cálculo recursivo de Pi usando `parallel for` e meça o tempo de execução. Execute o código pelo menos 3 vezes com diferentes valores de `MIN_BLK` e registre os tempos.
  
2. Paralelize o cálculo recursivo de Pi usando `#pragma omp task` e meça o tempo de execução. Execute o código pelo menos 3 vezes com diferentes números de tarefas e registre os tempos.

**Perguntas de Análise**:

- Qual abordagem (`parallel for` ou `tasks`) apresentou melhor desempenho? 

- O valor de `MIN_BLK` ou o número de tarefas influenciou significativamente o tempo de execução?

- Alguma abordagem teve variação maior entre execuções? Por quê?


### 3. Manipulação de Efeitos Colaterais no Vetor

1. Paralelize o código que modifica um vetor com `#pragma omp critical` para evitar acessos simultâneos ao vetor e registre os tempos de execução. Execute o código pelo menos 3 vezes e registre os tempos.

2. Modifique o código para pré-alocar a memória do vetor, evitando o uso de `push_back`, e meça o tempo. Execute o código pelo menos 3 vezes e registre os tempos.

**Perguntas de Análise**:

- Qual abordagem teve melhor desempenho: `omp critical` ou pré-alocação de memória?

- O uso de `omp critical` adicionou muito overhead? Como você pode justificar isso?

- A ordem dos dados no vetor foi mantida em ambas as abordagens?


### 4. Conclusão

- Resuma as principais conclusões com base nos resultados obtidos nos testes.

- Qual abordagem geral você considera mais eficiente para problemas recursivos e com efeitos colaterais?

- Alguma técnica apresentou resultados inesperados? O que poderia explicar isso?

