# Aula 10: Iteradores e funções customizadas 

Já vimos que, para valer a pena usar a GPU, precisamos tratar grandes volumes de dados ou realizar cálculos complexos. Hoje, nosso foco será calcular a variância e analisar as variações diárias de ações de maneira otimizada, usando iteradores dinâmicos e funções customizadas.

## Cálculo da Variância usando Iteradores Dinâmicos

Vamos calcular a variância das diferenças entre os preços de ações. A fórmula da variância é dada por:

\[
\text{Variância} = \frac{1}{n} \sum_{i=0}^{n} (x_i - \mu)^2
\]

onde \( \mu \) é a média. Sugerimos que você consulte a documentação de [thrust::constant_iterator](https://docs.nvidia.com/cuda/archive/10.1/pdf/Thrust_Quick_Start_Guide.pdf) e utilize este iterador para gerar dinamicamente o vetor de médias durante o cálculo, evitando assim o uso excessivo de memória. Isso permite calcular a variância sem precisar alocar espaço adicional na GPU.

## Análise de Variação Diária dos Preços de Ações

Em seguida, vamos trabalhar com o arquivo [stocks-google.txt](../10-thrust/files/stocks-google.txt). A ideia é criar um vetor que contenha a diferença diária entre o preço de um dia e o anterior. Se o vetor original se chama `stocks`, o vetor de saída `ganho_diario` será calculado de forma que:

\[
\text{ganho\_diario}[i] = \text{stocks}[i+1] - \text{stocks}[i]
\]

Lembre-se que o tamanho de `ganho_diario` será um elemento a menos que `stocks`. Utilize a operação [thrust::transform](https://docs.nvidia.com/cuda/archive/10.1/pdf/Thrust_Quick_Start_Guide.pdf) para calcular essas diferenças, prestando atenção nas condições de tamanho dos vetores.

## Contagem de Dias com Aumento no Preço das Ações

Após calcular as diferenças diárias, vamos descobrir quantas vezes o preço das ações subiu. Para isso, utilize a função [thrust::count_if](https://nvidia.github.io/cccl/thrust/api/function_group__counting_1gae2f8874093d33f3f0f49b51d8b26438c.html) com uma função customizada para contar apenas os elementos positivos de `ganho_diario`. 

## Cálculo do Aumento Médio nos Dias em que o Preço Subiu

Com a contagem em mãos, vamos calcular o aumento médio, mas apenas nos dias em que o preço subiu. Primeiro, você deve substituir todos os valores negativos de `ganho_diario` por zero, usando a função [thrust::replace_if](https://nvidia.github.io/cccl/thrust/api/function_group__replacing_1ga1e1d314818b9b40f1275b5f55c63c051.html). Depois, calcule a soma desses valores para obter o total dos aumentos, e divida pelo número de aumentos, obtido anteriormente com `count_if`. Isso permitirá calcular a média apenas dos valores positivos, filtrando os dias em que o preço das ações realmente aumentou.

## Discussão de Resultados

Ao longo desses exercícios, queremos que você reflita sobre a eficiência de usar a GPU para esses cálculos. Observe o tempo de execução e o uso de memória, e considere como iteradores dinâmicos e funções de transformação ajudam a otimizar o processamento. 

**A atividade deve ser entregue via BlackBoard até as 23h59 de 20/09** 