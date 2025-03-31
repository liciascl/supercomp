# Pi pelo método de Monte Carlo
Na aula de hoje iremos trabalhar com um algoritmo de sorteios aleatórios para calcular o pi. Ele é baseado em uma técnica de Otimização, Simulação e Estimação Paramétrica chamada [Monte Carlo](https://en.wikipedia.org/wiki/Monte_Carlo_method). Um bom vídeo para compreender essa técnica, [está disponível aqui](https://www.youtube.com/watch?v=7ESK5SaP-bc).

![pi](Pi_30K.gif)


O método consiste em sorteios aleatórios de pontos em um quadrado, que a proporção de pontos que caem dentro de um círculo inscrito nesse quadrado é usada para aproximar o valor de pi. 

\[
\pi \approx 4 \times \frac{\text{número de pontos dentro do círculo}}{\text{número total de pontos}}
\]


### Parte 1: Implementação Sequencial

Implemente a **versão sequencial do algoritmo de Monte Carlo**. Escreva um código C++ que gera \(N = 100000\) pontos aleatórios, calcule quantos caem dentro do círculo, e estime o valor de pi. 

Após implementar, **execute o programa** e anote os resultados. No seu relatório, inclua:

- O valor de pi estimado.

- O tempo de execução. 

**Reflexão**: No relatório, reflita sobre a implementação. A sequência de números aleatórios foi gerada corretamente? Houve alguma dificuldade na implementação?


### Parte 2: Primeira Tentativa de Paralelização

Agora que você tem a versão sequencial, tente **paralelizar o algoritmo**. Use a técnica de `for` paralelo para distribuir o sorteio de pontos entre várias threads. A paralelização, neste caso, deve focar em garantir que várias threads possam trabalhar simultaneamente no sorteio dos pontos, enquanto a variável `sum` (número de pontos dentro do círculo) é tratada com uma operação de **redução** para evitar conflitos entre threads.


**Reflexão**: Discuta no relatório por que a geração de números aleatórios pode ser um obstáculo em um ambiente paralelo? Como você resolveu esse problema? A sua solução impactou o desempenho do código?

Depois de paralelizar o código, rode e compare com a versão sequencial. No seu relatório, inclua:

- O valor de pi estimado pela versão paralela.

- O tempo de execução da versão paralela.


### Parte 3: Melhorando a Paralelização

Agora, vamos **melhorar a paralelização** abordando o problema da geração de números aleatórios de uma forma mais eficiente. Em vez de usar `omp critical` para proteger o sorteio de números aleatórios (o que pode ser lento), você pode dar a cada thread o seu próprio gerador de números aleatórios.

Para isso, baseie o [*seed*](https://en.cppreference.com/w/cpp/numeric/random) de cada gerador de números no índice da iteração ou em outra variável única para cada thread. Isso evita que os números sorteados sejam repetidos.

**Reflexão**: No relatório, discuta os efeitos dessa mudança. A geração de números aleatórios foi paralelizada de maneira eficaz? O valor de pi mudou de forma significativa? Houve melhoria no tempo de execução? Registre suas observações.

Depois de executar essa nova versão do algoritmo, inclua no seu relatório:

- O valor de pi estimado com essa melhoria.

- O tempo de execução da nova versão paralela.


### Conclusão e Comparação

No final do seu relatório, inclua uma **tabela comparando** o valor de pi e o tempo de execução de cada versão (sequencial, primeira tentativa paralela, e segunda tentativa paralela). Depois, resposta as perguntas a seguir: 

    1.  Houve uma melhoria significativa no tempo de execução entre a versão sequencial e as versões paralelas?

    2.  A estimativa de pi permaneceu precisa em todas as versões?

    3.  Quais foram os maiores desafios ao paralelizar o algoritmo, especialmente em relação aos números aleatórios?

    4.  O uso de threads trouxe benefícios claros para este problema específico?


