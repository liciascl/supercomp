# Aula 11: Funções Customizadas e Otimização com Fusion Kernel 

Nesta aula, vamos criar **operações customizadas** usando **functors**, entender como combinar transformações e reduções, e explorar a **otimização com fusion kernel**, uma técnica que melhora o desempenho ao combinar várias operações em uma única execução na GPU.


### Introdução a Operações Customizadas

A principal vantagem de trabalhar com Thrust é que podemos criar nossas próprias operações customizadas, conhecidas como **functors**. Functors nada mais são do que structs ou classes que sobrecarregam o operador `()` para aplicar uma função sobre os elementos de um vetor. Vamos começar com um exemplo clássico, o cálculo **Saxpy**.

O Saxpy (Single precision A X Plus Y) calcula a seguinte fórmula para cada elemento dos vetores `x` e `y`: 

\[
z = a \cdot x + y
\]

onde `a` é uma constante e `x` e `y` são vetores. Podemos implementar o Saxpy como um functor da seguinte maneira:

```cpp
struct saxpy
{
    int a;    
    saxpy(int a_) : a(a_) {};  // Construtor que inicializa a constante 'a'

    __host__ __device__
    double operator()(const int& x, const int& y) const {
        return a * x + y;  // Operação que será aplicada a cada elemento de x e y
    }
};
```

Criamos um struct chamado `saxpy` que recebe a constante `a` no construtor. Dentro do functor, sobrecarregamos o operador `()` para aplicar a fórmula \( a \cdot x + y \). A anotação `__host__ __device__` indica que essa operação pode ser executada tanto na CPU quanto na GPU.

Uma vez criado o functor, podemos usá-lo em uma função de transformação que processa dois vetores e combina os elementos de forma paralela. Isso é feito com a função `thrust::transform`:

```cpp
thrust::transform(d_a.begin(), d_a.end(), d_b.begin(), d_c.begin(), saxpy(m));
```

Aqui, `d_a` e `d_b` são os vetores de entrada, e `d_c` é o vetor de saída. A operação definida pelo functor `saxpy(m)` será aplicada a cada par de elementos de `d_a` e `d_b`. Esse tipo de operação é chamado de **transformação binária**, pois estamos combinando dois vetores.

Após a transformação, podemos copiar o vetor `d_c` de volta para a CPU, e o resultado final de cada operação será armazenado e exibido.

#### Exercício Prático:
Implemente o cálculo Saxpy para dois vetores aleatórios, experimente diferentes valores para a constante `a` e observe os resultados.

### Transformações Unárias e Binárias

Nem todas as transformações operam sobre dois vetores. Algumas vezes, queremos realizar uma operação em apenas um vetor, o que chamamos de **transformação unária**. Um exemplo seria elevar cada elemento de um vetor ao quadrado. Para isso, usamos um functor diferente, que aceita apenas um argumento:

```cpp
struct square
{
    __host__ __device__
    float operator()(const float& x) const {
        return x * x;  // Elevar ao quadrado
    }
};
```

Com esse functor, podemos aplicar a função `thrust::transform` para processar cada elemento de um vetor:

```cpp
thrust::transform(d_v.begin(), d_v.end(), d_v.begin(), square());
```

Aqui, `d_v` é tanto o vetor de entrada quanto de saída. Cada elemento de `d_v` será elevado ao quadrado.

Agora que entendemos a ideia de transformar vetores, podemos explorar um problema mais avançado: **calcular a magnitude de um vetor**.


### Calculando a Magnitude de um Vetor

A magnitude (ou norma) de um vetor \( v \) é uma medida de seu comprimento no espaço. A fórmula para a magnitude é:

\[
\text{magnitude}(v) = \sqrt{\sum{v_i^2}}
\]

Podemos implementar esse cálculo usando dois passos: primeiro, elevamos cada elemento ao quadrado (com uma transformação unária); depois, somamos os resultados (usando uma redução). Em vez de aplicar essas operações separadamente, podemos usar `thrust::transform_reduce`, que combina a transformação e a redução em uma única operação.

Um exemplo de como implementar isso:

```cpp
struct square
{
    __host__ __device__
    float operator()(const float& x) const {
        return x * x;  // Função de elevação ao quadrado
    }
};

float magnitude(thrust::device_vector<float>& v) {
    float sum_of_squares = thrust::transform_reduce(v.begin(), v.end(), square(), 0.0f, thrust::plus<float>());
    return std::sqrt(sum_of_squares);
}
```

O `thrust::transform_reduce` faz duas coisas: aplica o functor `square` para transformar os elementos e depois usa `thrust::plus<float>()` para somar os resultados. O valor inicial da soma é `0.0f`.

Neste exemplo, calculamos a soma dos quadrados dos elementos do vetor e, em seguida, aplicamos a função `std::sqrt()` para obter a magnitude.


#### Exercício Prático:
Implemente uma função que calcula a magnitude de um vetor de floats. Compare os resultados com uma implementação em C++ feita na CPU para ver a diferença no tempo de execução.


### Otimização com Fusion Kernel

Muitas vezes, quando executamos operações na GPU, estamos gastando mais tempo do que o necessário movendo dados entre a CPU e a GPU, ou entre diferentes partes da GPU. Isso acontece quando realizamos várias operações separadas que poderiam ser combinadas em uma única etapa. A técnica chamada **fusion kernel** resolve esse problema ao combinar múltiplas operações em um único kernel, economizando tempo.

Em vez de calcular a magnitude de um vetor com duas operações (uma transformação para elevar ao quadrado e outra para somar), podemos fazer isso com uma única chamada de função, combinando tudo com `thrust::transform_reduce`. A fusão de kernels elimina a necessidade de mover dados entre diferentes operações.

Para ilustrar o conceito, vamos calcular a variância de um conjunto de dados. A fórmula para a variância é:

\[
\text{Var}(X) = \frac{1}{n} \sum_{i=1}^{n} (x_i - \mu)^2
\]

onde \( \mu \) é a média do vetor. Em vez de calcular a média e, em seguida, calcular a variância, podemos usar o fusion kernel para fazer tudo em uma única etapa.

Um exemplo de como implementar isso:

```cpp
struct variance_op
{
    float mean;
    variance_op(float mean_) : mean(mean_) {}  // Construtor que armazena a média

    __host__ __device__
    float operator()(const float& x) const {
        float diff = x - mean;  // Diferença em relação à média
        return diff * diff;  // Elevar ao quadrado a diferença
    }
};

float calculate_variance(const thrust::device_vector<float>& d_vec, float mean) {
    return thrust::transform_reduce(d_vec.begin(), d_vec.end(), variance_op(mean), 0.0f, thrust::plus<float>()) / d_vec.size();
}
```

Neste código, usamos `thrust::transform_reduce` para calcular a soma das diferenças ao quadrado em uma única etapa. A operação `variance_op` subtrai a média de cada elemento e eleva o resultado ao quadrado, e `thrust::plus<float>()` soma os resultados. Tudo isso ocorre em uma única chamada de kernel, sem necessidade de transferir os dados entre a CPU e a GPU repetidamente.

#### Exercício Prático:
Implemente o cálculo da variância usando a técnica de fusion kernel. Compare o desempenho com a implementação que calcula a média e a variância em etapas separadas. Use diferentes tamanhos de vetor e observe as diferenças de desempenho.


**A atividade deve ser entregue via BlackBoard até as 23h59 de 24/09** 