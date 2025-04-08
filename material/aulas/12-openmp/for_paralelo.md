
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

