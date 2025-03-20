# Thrust para programação paralela em GPU

### O que é Thrust?

Thrust é uma biblioteca desenvolvida pela Nvidia para facilitar a programação em GPUs. Ela é inspirada na [biblioteca da STL (Standard Template Library)](https://www.geeksforgeeks.org/the-c-standard-template-library-stl/) do C++, mas com um foco em operações paralelas, tanto na GPU quanto na CPU. Thrust permite que o codigo seja executado de forma eficiente em paralelo, sem a complexidade de escrever kernels CUDA.

### Como Thrust Funciona?

Thrust fornece uma série de algoritmos e estruturas de dados otimizadas para execução em paralelo. Isso inclui:

- **Vetores**: `thrust::host_vector` e `thrust::device_vector` são vetores que gerenciam memória na CPU e GPU, respectivamente.
- **Algoritmos**: Thrust oferece uma variedade de algoritmos paralelos prontos para uso, como `thrust::sort`, `thrust::reduce`, `thrust::transform`, entre outros.
- **Iteradores**: Assim como a STL, Thrust usa iteradores para percorrer elementos de vetores, permitindo aplicar algoritmos de maneira eficiente.

Por exemplo, em vez de escrever um kernel CUDA manualmente para somar os elementos de um vetor, você pode usar a função `thrust::reduce`, que faz isso de forma otimizada e em paralelo na GPU.

### Vantagens de Usar Thrust

**1. Simplicidade:**
Thrust abstrai a complexidade da programação paralela em CUDA. 

**2. Portabilidade:**
Um dos grandes benefícios da Thrust é que o mesmo código pode ser compilado para rodar tanto na CPU quanto na GPU. Isso é possível porque Thrust permite escolher o backend de execução: você pode usar [OpenMP](http://homepages.math.uic.edu/~jan/mcs572f16/mcs572notes/lec09.html), [TBB (Threading Building Blocks)](https://oneapi-src.github.io/oneTBB/) ou a própria GPU. Isso torna seu código mais flexível e portátil.

**3. Desempenho:**
Thrust é otimizada para aproveitar ao máximo a capacidade de processamento paralelo das GPUs da Nvidia. Como muitos dos algoritmos oferecidos pela Thrust são implementados diretamente em CUDA, eles podem ser significativamente mais rápidos que implementações sequenciais tradicionais, especialmente em operações que envolvem grandes volumes de dados.

**4. Reuso de Código:**
Assim como a STL, Thrust permite reutilizar muito código. Como ela é baseada em templates, você pode aplicar os mesmos algoritmos em diferentes tipos de dados e vetores, sem precisar reescrever a lógica para cada caso.

### **Usando o Google Colab**

Primeiro, acesse o [Google Colab](https://colab.research.google.com/), crie um novo notebook. No Colab, podemos escolher o tipo de hardware que o nosso código vai usar. Para isso, vá no menu "Runtime" e clique em "change runtime type". Na seção “Hardware accelerator”, selecione "T4 GPU". Para que possamos executar o nosso código diretamente na GPU.

Para verificar se a GPU está ativa, rode o seguinte comando na célula do Colab:

```python
!nvidia-smi
```

Este comando vai mostrar informações sobre a GPU que estamos usando, como o modelo e o uso de memória. Se tudo estiver certo, você verá a GPU listada, o que confirma que o ambiente está pronto.

### **Compilação com NVCC**

Agora que o Colab está configurado, precisamos entender como compilar nosso código para rodar na GPU. O compilador que usamos para isso é o `nvcc`, que faz parte do pacote de ferramentas da Nvidia do CUDA Toolkit.

O `nvcc` é um compilador que identifica quais partes do código devem rodar na GPU e quais partes ficam na CPU. A grande vantagem dele é que, no final, tudo é combinado em um único executável que gerencia automaticamente quando a GPU ou CPU devem ser usadas.

Vamos compilar um exemplo simples para testar o `nvcc`. Carreque o arquivo [exemplo1.cu](files/exemplo1.cu) no colab, compile e crie um executável, usando o comando:

```bash
%%writefile exemplo1.cu
#include <thrust/device_vector.h> // Inclui a biblioteca para vetores na GPU (device)
#include <thrust/host_vector.h>   // Inclui a biblioteca para vetores na CPU (host)
#include <iostream>               // Biblioteca padrão para entrada e saída de dados

int main() {
    // Cria um vetor na CPU (host) com 5 elementos, todos inicializados com 0
    thrust::host_vector<double> host(5, 0);
    host[4] = 35; // Altera o último elemento do vetor para 35

    /* Os dados do vetor 'host' são copiados para a GPU,
       criando um vetor equivalente na GPU (device) */
    thrust::device_vector<double> dev(host);

    /* Altera o vetor na CPU, mas não afeta o vetor na GPU,
       pois a cópia já foi feita anteriormente */
    host[2] = 12; // Altera o terceiro elemento do vetor na CPU para 12

    // Exibe os elementos do vetor na CPU
    printf("Host vector: ");
    for (auto i = host.begin(); i != host.end(); i++) {
        std::cout << *i << " "; // Acesso rápido aos elementos na CPU
    }
    printf("\n");

    // Exibe os elementos do vetor na GPU
    printf("Device vector: ");
    for (auto i = dev.begin(); i != dev.end(); i++) {
        std::cout << *i << " "; // Acesso aos elementos na GPU é lento, pois os dados estão na GPU
    }
    printf("\n");
}

```

```bash
!nvcc -arch=sm_75 -std=c++14 exemplo1.cu -o exemplo1
```

Se tudo funcionar corretamente, este comando vai gerar um executável chamado `exemplo1`, que podemos rodar para ver os resultados.

```bash
!./exemplo1
```
### Transformações e Operações Matemáticas com Vetores na GPU

Uma das grandes vantagens de Thrust é a capacidade de aplicar transformações e operações ponto a ponto de forma eficiente, diretamente na GPU. Isso é extremamente útil para manipulações de vetores, onde queremos aplicar operações como soma, multiplicação, etc, de forma paralela.

Com Thrust, podemos facilmente realizar essas transformações entre vetores, utilizando funções como `thrust::transform`, que permite aplicar uma operação matemática entre os elementos de dois vetores ou transformar um vetor usando um valor constante. 

Neste exemplo, vamos criar dois vetores na GPU e aplicar transformações entre eles, demonstrando como podemos somar os elementos de um vetor com outro e preencher vetores com valores específicos. Essas operações são realizadas de maneira otimizada na GPU, aproveitando o paralelismo do hardware para acelerar o processamento.

```cpp
%%writefile exemplo2.cu
#include <thrust/device_vector.h>              // Inclui a biblioteca para vetores na GPU
#include <thrust/host_vector.h>                // Inclui a biblioteca para vetores na CPU
#include <thrust/sequence.h>                   // Inclui a função para preencher vetores com uma sequência de números
#include <thrust/functional.h>                 // Inclui operações matemáticas padrão, como soma e multiplicação
#include <thrust/transform.h>                  // Inclui a função para transformar vetores com operações ponto a ponto
#include <thrust/iterator/constant_iterator.h> // Inclui o iterador constante necessário
#include <iostream>   

int main() {
    // Cria um vetor na GPU (device_vector) com 10 elementos, todos inicializados com 0
    thrust::device_vector<double> V1(10, 0);
    // Preenche V1 com uma sequência de números: {0, 1, 2, ..., 9}
    thrust::sequence(V1.begin(), V1.end());

    // Cria um vetor na GPU com 5 elementos, todos inicializados com 0
    thrust::device_vector<double> V2(5, 0);
    // Preenche os dois primeiros elementos de V2 com 5.5: {5.5, 5.5, 0, 0, 0}
    thrust::fill(V2.begin(), V2.begin() + 2, 5.5);
    // Preenche os elementos restantes de V2 com 10: {5.5, 5.5, 10, 10, 10}
    thrust::fill(V2.begin() + 2, V2.end(), 10);

    // Cria dois vetores na GPU com 10 elementos, para armazenar resultados das operações
    thrust::device_vector<double> V3(10); // Vetor para armazenar o resultado da soma
    thrust::device_vector<double> V4(10); // Vetor para armazenar o resultado da multiplicação

    // Aplica a operação de soma elemento por elemento entre V1 e V2 e armazena o resultado em V3
    // Como V2 tem menos elementos, o restante de V1 é somado com zeros (elemento padrão).
    // Resultado: V3 = {0+5.5, 1+5.5, 2+10, 3+10, 4+10, 5+0, ..., 9+0}
    thrust::transform(V1.begin(), V1.end(), V2.begin(), V3.begin(), thrust::plus<double>());

    // Aplica multiplicação de V1 com o valor constante 0.5
    // e armazenaria o resultado em V4.
    thrust::transform(V1.begin(), V1.end(), thrust::constant_iterator<double>(0.5), V4.begin(), thrust::multiplies<double>());

    // Imprime os elementos de V1
    printf("V1: ");
    for (thrust::device_vector<double>::iterator i = V1.begin(); i != V1.end(); i++) {
        std::cout << *i << " "; // Acessa e imprime cada elemento do vetor V1
    }
    printf("\n");

    // Imprime os elementos de V2
    printf("V2: ");
    for (thrust::device_vector<double>::iterator i = V2.begin(); i != V2.end(); i++) {
        std::cout << *i << " "; // Acessa e imprime cada elemento do vetor V2
    }
    printf("\n");

    // Imprime os elementos de V3
    printf("V3: ");
    for (thrust::device_vector<double>::iterator i = V3.begin(); i != V3.end(); i++) {
        std::cout << *i << " "; // Acessa e imprime cada elemento do vetor V3
    }
    printf("\n");

    // Imprime o vetor V4, que foi criado mas não utilizado.
     printf("V4: ");
    for (thrust::device_vector<double>::iterator i = V4.begin(); i != V4.end(); i++) {
         std::cout << *i << " ";
    }
    printf("\n");

    return 0;
}


```
Para compilar o exemplo2 no colab

```
!nvcc -arch=sm_75 -std=c++14 exemplo2.cu -o exemplo2
```

Para executar o binário

```
!./exemplo2
```


### Leitura de arquivos pela GPU

Neste exemplo,o código faz a leitura de uma série de valores numéricos que representam, por exemplo, os preços históricos de ações. Esses valores são inicialmente armazenados em um vetor na memória da CPU, chamado `host_vector`. Em seguida, esses dados são copiados para a GPU usando um `device_vector`.

O objetivo do código é medir dois tempos: o tempo de leitura dos dados na CPU e o tempo de cópia dos dados para a GPU. Avaliar esses tempos ajuda a identificar como a comunicação entre a CPU e a GPU pode afetar o desempenho geral do seu programa. 

```cpp
%%writefile stocks.cu
#include <thrust/device_vector.h>   // Inclui a biblioteca Thrust para vetores na GPU
#include <thrust/host_vector.h>     // Inclui a biblioteca Thrust para vetores na CPU
#include <iostream>                 // Biblioteca padrão para entrada e saída de dados
#include <chrono>                   // Biblioteca para medir o tempo de execução do código
using namespace std;

int main() {
    int n = 2518;                   // Define o tamanho do vetor (2518 elementos)
    double value = 0.0;             // Variável auxiliar para armazenar o valor lido da entrada
    std::chrono::duration<double> diff; // Variável para armazenar a diferença de tempo calculada

    // Marca o início da medição do tempo de leitura dos dados
    auto leitura_i = std::chrono::steady_clock::now();

    // Cria um vetor na CPU (host_vector) com n elementos, todos inicializados com 0
    thrust::host_vector<double> host(n, 0);

    // Loop para ler n valores da entrada padrão e armazená-los no vetor host
    for (int i = 0; i < n; i++) {
        cin >> value;              // Lê um valor da entrada padrão
        host[i] = value;           // Armazena o valor lido no vetor host
    }

    // Marca o fim da medição do tempo de leitura dos dados
    auto leitura_f = std::chrono::steady_clock::now();

    // Calcula o tempo gasto na leitura dos dados
    diff = leitura_f - leitura_i;

    // Exibe o tempo de leitura em segundos
    cout << "Tempo de LEITURA (em segundos)  " << diff.count() << endl;

    // Marca o início da medição do tempo de cópia dos dados da CPU para a GPU
    auto copia_i = std::chrono::steady_clock::now();

    // Cria um vetor na GPU (device_vector) copiando os dados do vetor host da CPU
    thrust::device_vector<double> dev(host);

    // Marca o fim da medição do tempo de cópia dos dados
    auto copia_f = std::chrono::steady_clock::now();

    // Calcula o tempo gasto na cópia dos dados para a GPU
    diff = copia_f - copia_i;

    // Exibe o tempo de cópia em segundos
    cout << "Tempo de CÓPIA (em segundos)  " << diff.count() << endl;

    return 0;
}

```

Para compilar o código:

```
!nvcc -arch=sm_70 -std=c++14 stocks.cu -o stocks
```
Faça upload [deste arquivo](files/stocks-google.txt) no seu colab para executar o exemplo

```
!./stocks < stocks-google.txt

```


### **Operações de Redução e Transformações**

Além de gerenciar a transferência de dados, a Thrust também facilita operações complexas como reduções (soma, máximo, mínimo) e transformações entre vetores. Esses tipos de operações são muito úteis em cálculos intensivos, como análise de séries temporais de preços de ações.

**Reduções com Thrust:**

Reduções são operações que reduzem um vetor a um único valor. Por exemplo, podemos calcular a soma, o máximo ou a média dos elementos de um vetor. A função `thrust::reduce` é usada para esse tipo de operação.

Um exemplo para calcular a soma dos elementos de um vetor:

```cpp
#include <thrust/device_vector.h>
#include <thrust/reduce.h>
#include <iostream>

int main() {
    // Criando um vetor na GPU com 5 elementos: {10, 20, 30, 40, 50}
    thrust::device_vector<int> vec_gpu(5);
    thrust::sequence(vec_gpu.begin(), vec_gpu.end(), 10, 10);

    // Calculando a soma dos elementos
    int soma = thrust::reduce(vec_gpu.begin(), vec_gpu.end(), 0, thrust::plus<int>());

    std::cout << "Soma dos elementos: " << soma << std::endl; // Saída: 150

    return 0;
}
```

Aqui, usamos `thrust::sequence` para preencher o vetor e `thrust::reduce` para somar todos os elementos. O valor `0` é o valor inicial da soma e `thrust::plus<int>` é a operação de adição.

**Transformações entre Vetores:**

Além das reduções, podemos realizar transformações entre vetores, que envolvem modificar um vetor com base em operações ponto a ponto, ou até mesmo combinar dois vetores em um.

Por exemplo, vamos calcular a diferença ponto a ponto entre os preços das ações da Apple e da Microsoft usando `thrust::transform`:

```cpp
#include <thrust/device_vector.h>
#include <thrust/transform.h>
#include <iostream>

int main() {
    // Criando vetores de preços na GPU
    thrust::device_vector<double> AAPL(5, 150.0);  // Exemplo com preço fixo
    thrust::device_vector<double> MSFT(5, 140.0);  // Exemplo com preço fixo
    thrust::device_vector<double> diff(5);         // Para armazenar as diferenças

    // Calculando a diferença ponto a ponto entre os preços
    thrust::transform(AAPL.begin(), AAPL.end(), MSFT.begin(), diff.begin(), thrust::minus<double>());

    // Exibindo as diferenças
    for (int i = 0; i < diff.size(); i++) {
        std::cout << "Diferença " << i << ": " << diff[i] << std::endl;
    }

    return 0;
}
```

Neste exemplo, `thrust::minus<double>` é a operação usada para calcular a diferença entre os elementos correspondentes dos dois vetores.

Para entender mais sobre as funcionalidades da Thrust, verifique a documentação oficial disponível no [site da Thrust](https://thrust.github.io/doc/modules.html). 



### Atividade

Vamos trabalhar com um arquivo que contém os preços das ações do Google nos últimos 10 anos, [stocks-google.txt](files/stocks-google.txt). A ideia é ler esses dados com um `thrust::host_vector`, transferir para a GPU e, em seguida, processá-los. 

**Exercício 1: Leitura e Transferência de Dados**

1. Leia os preços das ações de [stocks-google.txt](files/stocks-google.txt) e armazenem em um `host_vector`.
2. Crie um `device_vector` e transfiram os dados do `host_vector` para ele.
3. Meça o tempo de alocação e cópia dos dados usando a biblioteca `std::chrono` e imprimam esse tempo para ver quanto tempo a operação leva.

**Exercício 2: Cálculo de Médias e Extremos**

Agora que os dados estão na GPU, vamos realizar algumas operações de redução. As operações de redução são aquelas que transformam um vetor em um único valor, como calcular a soma ou o máximo dos elementos.

1. Calcule o preço médio das ações do Google nos últimos 10 anos.
2. Calcule o preço médio das ações nos últimos 365 dias.
3. Encontre o maior e o menor preço do período total e do último ano.

**Exercício 3: Transformações entre Vetores**

Vamos trabalhar com outro arquivo, [stocks2.txt](files/stocks2.txt), que contém os preços das ações da Apple e Microsoft. O objetivo é calcular a diferença média entre os preços das ações das duas empresas.

O dataset contém preços simulados das ações da Apple (AAPL) e da Microsoft (MSFT) ao longo dos últimos 10 anos, com cerca de 3520 registros, representando aproximadamente 352 dias de negociação por ano. Cada linha do arquivo tem dois valores separados por vírgula: o primeiro é o preço da ação da Apple e o segundo é o preço da ação da Microsoft.


1. Leia os preços das ações de ambas as empresas e armazenem em dois `device_vectors`.
2. Calcule a diferença ponto a ponto entre os dois vetores e armazenem as diferenças em um terceiro vetor.
3. Calcule a média das diferenças para entender qual empresa teve, em média, um preço maior.

**Entrega do relatório para 24/03 as 23h59**