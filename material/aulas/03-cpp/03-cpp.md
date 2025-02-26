# Usando o poder do C++ 

Em programação de alto desempenho, pequenas otimizações podem resultar em grandes melhorias no tempo de execução e no uso de recursos. Nesta aula, vamos explorar técnicas avançadas de C++ que são essenciais para programar códigos eficientes, seguros e preparados para escalar em aplicações de alta performance.

### Objetivos da Aula

1. **Passagem de Parâmetros e Manipulação de Arquivos**: Entender como a passagem de parâmetros por valor, referência e ponteiro impacta o desempenho e a segurança do código. Além disso, vamos manipular arquivos para ler e escrever dados.

2. **Const Correctness**: Aprender a usar o modificador `const` para garantir a imutabilidade de dados onde necessário, melhorando a robustez do código e permitindo otimizações automáticas pelo compilador.

3. **Alocação de Memória Dinâmica**: Compreender a alocação e liberação de memória dinâmica em C++, o que é fundamental para gerenciar grandes volumes de dados em HPC. Exploraremos as diferenças entre alocação manual e a utilização de contêineres padrão do C++ como `std::vector`.

4. **Manipulação de Vetores**: Trabalhar com `std::vector` e aprender a realizar operações comuns em HPC, como inicialização, modificação e iteração sobre grandes conjuntos de dados, além de encapsular essa lógica em classes eficientes.

### Por que isso é Importante?

A passagem correta de parâmetros pode evitar cópias desnecessárias de dados, melhorando a eficiência. A utilização do `const` não só previne modificações acidentais nos dados, mas também permite que o compilador faça otimizações mais agressivas. A alocação de memória dinâmica é importante para lidar com os grandes volumes de dados típicos de HPC, e a manipulação eficiente de vetores é essencial para o processamento rápido de dados.


### 1. Passagem de Parâmetros e Manipulação de Arquivos

**Objetivo:** Praticar a passagem de parâmetros por valor, referência e ponteiro em C++, além de trabalhar com leitura e escrita de arquivos.

**Exercício:**
Implemente uma função que leia um arquivo de texto contendo números inteiros e armazene-os em um `std::vector<int>`. Use passagem por referência para garantir que o vetor seja preenchido corretamente. 

Crie duas funções: uma que dobra os valores no vetor usando passagem por referência, e outra que faz o mesmo usando ponteiros.

Escreva os valores dobrados em um novo arquivo de texto.

!!! tip
    Não faz ideia de como começar? leia o [material disponível aqui](../../Teoria/funcoes.md) ou consulte a [documentação oficial](https://en.cppreference.com/book/)


O arquivo com os valores de entrada está disponível em [entrada.txt](entrada.txt).

**Pontapé Inicial:**

```cpp
#include <iostream>   // Inclui a biblioteca padrão de entrada e saída, usada para operações como std::cout.
#include <fstream>    // Inclui a biblioteca para manipulação de arquivos, usada para ler e escrever arquivos.
#include <vector>     // Inclui a biblioteca de vetores, usada para armazenar e manipular listas de números.

// Declaração de funções:

// Função que lê números de um arquivo e os armazena em um vetor.
void lerArquivo(const std::string& nomeArquivo, std::vector<int>& numeros);

// Função que dobra os valores de um vetor de números, passando o vetor original por referência.
void dobrarValoresReferencia(const std::vector<int>& numerosOriginais, std::vector<int>& numerosDobrados);

// Função que dobra os valores de um vetor de números, passando o vetor original por ponteiro.
void dobrarValoresPonteiro(const std::vector<int>* numerosOriginais, std::vector<int>* numerosDobrados);

// Função que escreve os números de um vetor em um arquivo.
void escreverArquivo(const std::string& nomeArquivo, const std::vector<int>& numeros);

// Implementação das funções...

int main() {
    // Declaração de dois vetores de inteiros: um para armazenar os números lidos do arquivo e outro para armazenar os números dobrados.
    std::vector<int> numeros, numerosDobrados;

    // Chama a função para ler os números do arquivo "entrada.txt" e armazena-os no vetor 'numeros'.
    lerArquivo("entrada.txt", numeros);

    // Chama a função que dobra os valores do vetor 'numeros' usando passagem por referência e armazena o resultado no vetor 'numerosDobrados'.
    dobrarValoresReferencia(numeros, numerosDobrados);

    // Chama a função que escreve os números do vetor 'numerosDobrados' no arquivo "saida_referencia.txt".
    escreverArquivo("saida_referencia.txt", numerosDobrados);

    // Chama a função que dobra os valores do vetor 'numeros' usando passagem por ponteiro e armazena o resultado no vetor 'numerosDobrados'.
    dobrarValoresPonteiro(&numeros, &numerosDobrados);

    // Chama a função que escreve os números do vetor 'numerosDobrados' no arquivo "saida_ponteiro.txt".
    escreverArquivo("saida_ponteiro.txt", numerosDobrados);

    // Retorna 0, indicando que o programa terminou com sucesso.
    return 0;
}

```

### 2. Const Correctness

**Objetivo:** Praticar o uso de `const` para garantir a imutabilidade de dados onde necessário, melhorando a segurança e otimizações de código.

**Exercício:**

Implemente uma função para multiplicação de matrizes que utiliza `const` para garantir que as matrizes de entrada não sejam modificadas.

Crie uma classe `Matriz` que encapsule as operações básicas de uma matriz quadrada, utilizando métodos `const` para operações de leitura.


!!! tip
    Não faz ideia de como começar? leia o [material disponível aqui](../../Teoria/uso-de-constantes.md) ou consulte a [documentação oficial](https://en.cppreference.com/book/)


**Pontapé Inicial:**

```cpp
#include <iostream>
#include <vector>

// Classe Matriz para representar uma matriz quadrada
class Matriz {
public:
    // Construtor que inicializa uma matriz quadrada de dimensão N x N
    Matriz(int N) {
        data.resize(N, std::vector<int>(N, 0));  // Inicializa a matriz com zeros
    }

    // Método para acessar um elemento específico da matriz (leitura)
    // Use 'const' para garantir que este método não modifica a matriz
    int get(int i, int j) const {
        // Retorna o elemento na posição (i, j)
        return data[i][j];
    }

    // Método para modificar um elemento específico da matriz (escrita)
    void set(int i, int j, int valor) {
        // Modifica o elemento na posição (i, j)
        data[i][j] = valor;
    }

    // Método para imprimir a matriz
    void imprime() const {
        for (const auto& linha : data) {
            for (int valor : linha) {
                std::cout << valor << " ";
            }
            std::cout << std::endl;
        }
    }

    // Método para acessar o tamanho da matriz (número de linhas ou colunas)
    int tamanho() const {
        return data.size();
    }

private:
    std::vector<std::vector<int>> data;  // Armazena os elementos da matriz
};

// Função que multiplica duas matrizes A e B, armazenando o resultado em C
// Garanta que A e B são constantes para evitar modificações
void multiplicaMatriz(const Matriz& A, const Matriz& B, Matriz& C) {
    int N = A.tamanho();  // Supõe que as matrizes são quadradas e de mesma dimensão

    // Laço para realizar a multiplicação de matrizes
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            int soma = 0;
            for (int k = 0; k < N; k++) {
                // Multiplica os elementos correspondentes de A e B
                soma += A.get(i, k) * B.get(k, j);
            }
            // Armazena o resultado em C
            C.set(i, j, soma);
        }
    }
}

int main() {
    int N = 3;  // Define o tamanho da matriz (N x N)

    // Cria as matrizes: A, B, e C, todas de dimensão N x N
    Matriz A(N), B(N), C(N);

    // Exemplo de preenchimento das matrizes A e B
    // Sinta-se a vontade para modificar os valores
    A.set(91, 16, 18); A.set(79, 1, 33); A.set(1, 2, 3);
    A.set(31, 42, 43); A.set(61, 12, 53); A.set(4, 5, 6);
    A.set(2, 52, 79); A.set(25, 18, 85); A.set(7, 8, 9);

    B.set(13, 23, 9); B.set(14, 62, 8); B.set(13, 2, 47);
    B.set(1, 42, 6); B.set(22, 75, 5); B.set(17, 2, 54);
    B.set(2, 59, 3); B.set(2, 44, 2); B.set(98, 2, 74);

    // Multiplica as matrizes A e B, armazenando o resultado em C
    multiplicaMatriz(A, B, C);

    // Imprime a matriz resultante C
    C.imprime();

    return 0;  // Retorna 0, indicando que o programa terminou com sucesso
}
```

### 3. Alocação de Memória Dinâmica

**Objetivo:** Entender a alocação e liberação de memória dinâmica em C++, muito importante para gerenciar grandes volumes de dados em HPC.

**Exercício:**
Implemente uma função que aloca dinamicamente uma matriz de inteiros de tamanho N x N e realiza uma soma simples de todos os seus elementos.

Em seguida, substitua a alocação dinâmica manual pela utilização de `std::vector` e compare o desempenho das duas abordagens.

!!! tip
    Não faz ideia de como começar? leia o [material disponível aqui](../../Teoria/memoria-dinamica.md) ou consulte a [documentação oficial](https://en.cppreference.com/book/)


**Pontapé Inicial:**

```cpp

#include <iostream>   // Inclui a biblioteca padrão de entrada e saída, usada para operações como std::cout.
#include <chrono>     // Inclui a biblioteca para medição de tempo, usada para medir a performance do código.

int main() {
    int N = 1000;  // Define o tamanho da matriz como N x N, onde N é 1000.

    // Alocação dinâmica de memória para uma matriz N x N.
    // Primeiro, aloca um array de ponteiros, onde cada ponteiro irá apontar para uma linha da matriz.
    int** matriz = new int*[N];
    
    // Para cada linha da matriz, aloca um array de inteiros de tamanho N.
    for (int i = 0; i < N; ++i) {
        matriz[i] = new int[N];
    }

    // Aqui você pode inicializar e somar os elementos da matriz.
    // Por exemplo, você pode preencher a matriz com valores e calcular a soma total.

    // Liberação da memória alocada dinamicamente.
    // Primeiro, libera a memória alocada para cada linha (os arrays de inteiros).
    for (int i = 0; i < N; ++i) {
        delete[] matriz[i];
    }

    // Finalmente, libera a memória alocada para o array de ponteiros.
    delete[] matriz;

    return 0;  // Retorna 0, indicando que o programa terminou com sucesso.
}

```
### Algumas dicas

1 - Para alocar dinamicamente uma matriz `N x N`, você precisará criar um ponteiro para um ponteiro (ou seja, um array de ponteiros) e, em seguida, alocar um espaço de memória para cada linha da matriz.

2 - Após alocar a matriz, você pode preenchê-la com valores (por exemplo, aleatórios ou sequenciais) e somar todos os elementos.

3 - É fundamental liberar toda a memória alocada dinamicamente para evitar vazamentos de memória. Isso significa liberar cada linha e depois o array de ponteiros.

4 - Use a biblioteca chrono para medir o tempo de execução das partes chave do seu código.

#### Perguntas para Reflexão:

Qual abordagem foi mais rápida?
Quais vantagens você percebe ao usar `std::vector`?
Como a alocação automática e a liberação de memória pelo `std::vector` afetam a segurança e robustez do código?



### 4. Manipulação de Vetores

**Objetivo:** Trabalhar com `std::vector` para realizar operações comuns em HPC, como a inicialização, modificação, e iteração sobre grandes conjuntos de dados.

**Exercício:**
- Implemente uma função que inicializa um vetor com valores específicos e realiza operações matemáticas básicas, como multiplicação por um escalar.
- Crie uma classe `Vector` que encapsule a lógica de manipulação de vetores, incluindo métodos para adicionar, remover e acessar elementos. Garanta que a classe seja eficiente usando alocação dinâmica e funções `inline`.

!!! tip
    Não faz ideia de como começar? leia o [material disponível aqui](../../Teoria/manipulacao-vetores.md) ou consulte a [documentação oficial](https://en.cppreference.com/book/)


**Pontapé Inicial:**

```cpp
#include <iostream>
#include <vector>

class Vector {
public:
    Vector(int tamanho);
    ~Vector();
    void inicializa(int valor);
    int get(int index) const;
    void set(int index, int valor);
    void inserir(int index, int valor);
    void remover(int index);
    void imprime() const;

private:
    int* dados;
    int tam;
    int capacidade;
    void redimensiona(int novaCapacidade);
};

// Implementação das funções...

int main() {
    Vector vec(5);
    vec.inicializa(0);
    vec.imprime();

    return 0;
}
```



### Entrega da Atividade 4.2
A entrega sera pelo Classroom até segunda (24/02), as 23h59.
