# Aula 10 - Thrust para programação paralela em GPU

### O que é Thrust?

Thrust é uma biblioteca desenvolvida pela Nvidia para facilitar a programação de GPUs. Ela é inspirada na biblioteca da STL (Standard Template Library) do C++, mas com um foco em operações paralelas, tanto na GPU quanto na CPU. Thrust permite que o codigo seja executado de forma eficiente em paralelo, sem a complexidade de escrever kernels CUDA explícitos, que são necessários para programar diretamente em CUDA.

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
Um dos grandes benefícios da Thrust é que o mesmo código pode ser compilado para rodar tanto na CPU quanto na GPU. Isso é possível porque Thrust permite escolher o backend de execução: você pode usar OpenMP, TBB (Threading Building Blocks) ou a própria GPU, dependendo do hardware disponível. Isso torna seu código mais flexível e portátil.

**3. Desempenho:**
Thrust é otimizada para aproveitar ao máximo a capacidade de processamento paralelo das GPUs da Nvidia. Como muitos dos algoritmos oferecidos pela Thrust são implementados diretamente em CUDA, eles podem ser significativamente mais rápidos que implementações sequenciais tradicionais, especialmente em operações que envolvem grandes volumes de dados.

**4. Reuso de Código:**
Assim como a STL, Thrust permite reutilizar muito código. Como ela é baseada em templates, você pode aplicar os mesmos algoritmos em diferentes tipos de dados e vetores, sem precisar reescrever a lógica para cada caso.

### **Usando o Google Colab**

Primeiro, acesse o [Google Colab](https://colab.research.google.com/) e criem um novo notebook. No Colab, podemos escolher o tipo de hardware que o nosso código vai usar. Para isso, vá no menu "Ambiente de Execução" e clique em "Alterar tipo de ambiente de execução". Na seção “Acelerador de hardware”, selecione "GPU". Para que possamos executar o nosso código diretamente na GPU.

Para verificar se a GPU está ativa, rode o seguinte comando na célula do Colab:

```python
!nvidia-smi
```

Este comando vai mostrar informações sobre a GPU que estamos usando, como o modelo e o uso de memória. Se tudo estiver certo, você verá a GPU listada, o que confirma que o ambiente está pronto.

### **Compilação com NVCC**

Agora que o Colab está configurado, precisamos entender como compilar nosso código para rodar na GPU. O compilador que usamos para isso é o `nvcc`, que faz parte do pacote de ferramentas da Nvidia do CUDA Toolkit.

O `nvcc` é um compilador que identifica quais partes do código devem rodar na GPU e quais partes ficam na CPU. A grande vantagem dele é que, no final, tudo é combinado em um único executável que gerencia automaticamente quando a GPU ou CPU devem ser usadas.

Vamos compilar um exemplo simples para testar o `nvcc`. Carreque o arquivo [exemplo1-criacao-iteracao.cu](files/exemplo1-criacao-iteracao.cu) no colab, compile e crie um executável, usando o comando:


```bash
!nvcc -arch=sm_70 -std=c++14 exemplo1-criacao-iteracao.cu -o exemplo1
```

Se tudo funcionar corretamente, este comando vai gerar um executável chamado `exemplo1`, que podemos rodar para ver os resultados.

### **Transferência de Dados entre CPU e GPU**

Agora, vamos entender como os dados são transferidos entre CPU e GPU usando a biblioteca Thrust. A CPU e a GPU têm memórias separadas, o que significa que para processar dados na GPU, precisamos transferi-los da CPU para a GPU.

Com a Thrust, temos dois tipos de vetores: `host_vector`, que armazena dados na memória da CPU, e `device_vector`, que armazena dados na memória da GPU. Veja este exemplo de código:

```cpp
// exemplo1-criacao-iteracao.cu
#include <thrust/host_vector.h>
#include <thrust/device_vector.h>
#include <thrust/sequence.h>
#include <thrust/fill.h>
#include <iostream>

int main() {
    // Criação de um vetor na CPU (host)
    thrust::host_vector<int> host_vec(5, 0); // Inicia com {0, 0, 0, 0, 0}
    std::cout << "Host vector: ";
    for (int i = 0; i < host_vec.size(); i++) {
        std::cout << host_vec[i] << " ";
    }
    std::cout << std::endl;

    // Preenchendo o vetor com uma sequência: {0, 1, 2, 3, 4}
    thrust::sequence(host_vec.begin(), host_vec.end());

    // Criação de um vetor na GPU (device) e transferência dos dados da CPU para a GPU
    thrust::device_vector<int> device_vec = host_vec;

    // Modificando os dois primeiros elementos na GPU
    thrust::fill(device_vec.begin(), device_vec.begin() + 2, 13); // {13, 13, 2, 3, 4}

    // Transferindo os dados de volta da GPU para a CPU
    host_vec = device_vec;

    // Mostrando o resultado
    std::cout << "Device vector after modifications: ";
    for (int i = 0; i < host_vec.size(); i++) {
        std::cout << host_vec[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}

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


### **Explorando a Documentação da Thrust**

Para entender mais sobre as funcionalidades da Thrust, verifique a documentação oficial disponível no [site da Thrust](https://thrust.github.io/doc/modules.html). 



### Atividade 10

Vamos trabalhar com um arquivo que contém os preços das ações do Google nos últimos 10 anos, [stocks-google.txt](files/stocks-google.txt). A ideia é ler esses dados com um `thrust::host_vector`, transferir para a GPU e, em seguida, processá-los. 

**Exercício 1: Leitura e Transferência de Dados**

1. Leiam os preços das ações de [stocks-google.txt](files/stocks-google.txt) e armazenem em um `host_vector`.
2. Criem um `device_vector` e transfiram os dados do `host_vector` para ele.
3. Meçam o tempo de alocação e cópia dos dados usando a biblioteca `std::chrono` e imprimam esse tempo para ver quanto tempo a operação leva.

**Exercício 2: Cálculo de Médias e Extremos**

Agora que os dados estão na GPU, vamos realizar algumas operações de redução. As operações de redução são aquelas que transformam um vetor em um único valor, como calcular a soma ou o máximo dos elementos.

1. Calculem o preço médio das ações do Google nos últimos 10 anos.
2. Calculem o preço médio das ações nos últimos 365 dias.
3. Encontrem o maior e o menor preço do período total e do último ano.

**Exercício 3: Transformações entre Vetores**

Vamos trabalhar com outro arquivo, [stocks2.csv](files/stock2.csv), que contém os preços das ações da Apple e Microsoft. O objetivo é calcular a diferença média entre os preços das ações das duas empresas.

1. Leiam os preços das ações de ambas as empresas e armazenem em dois `device_vectors`.
2. Calculem a diferença ponto a ponto entre os dois vetores e armazenem as diferenças em um terceiro vetor.
3. Calculem a média das diferenças para entender qual empresa teve, em média, um preço maior.

**Submeta o link do seu repositório no BlackBoard até 17/09 as 23h59**