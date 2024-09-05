# Aula 09 - Heurísticas

## Mais caro primeiro

A ideia desta heurística é não deixar nenhum objeto valioso para trás! Por isso vamos ser ganaciosos e pegar **primeiro os objetos mais caros**! Se um objeto valioso não couber passamos para os mais baratos e prosseguimos até examinar todos objetos.

!!! question
    Qual é a complexidade computacional deste algoritmo? Ele é a melhor implementação possível?

    ??? details "Resposta"
        Se o algoritmo descrito em sua resposta anterior envolver ordenação, então ele tem complexidade $\mathcal{O}(n\log n)$ e é o melhor possível sim (você consegue explicar por que?). Se você fez um loop duplo que procura pelo maior a cada iteração então seu algoritmo é $\mathcal{O}(n^2)$.

!!! example
    Agora que temos um algoritmo, crie uma implementação do programa acima.

    **Dicas**:

    * C++ já possui um algoritmo de ordenação implementado no cabeçalho [`<algorithm>`](http://cplusplus.com/reference/algorithm/sort/). Use-o.
    * Busque por ordenação indireta para entender como ordenar os três vetores ao mesmo tempo.
    * Pode ser conveniente organizar os dados usando `struct`.

## Mais leve primeiro

Vamos testar uma abordagem oposta: **quantidade agora é o foco**. Por isso vamos ser práticos e pegar **o maior número de objetos possível**! Começaremos agora pelos objetos mais leves e vamos torcer para que a quantidade grande de objetos selecionados resulte em uma mochila com alto valor.

!!! question
    Compare esta heurística com a da seção anterior levando em conta a complexidade computacional.

!!! question
    Quais partes do programa da heurística anterior podem ser aproveitadas para implementar a descrita acima?

!!! example
    Implemente agora a heurística do mais leve. Chame seu programa de `mais_leve`, mantendo também o código do anterior.

## Analisando nossas heurísticas

!!! question
    Crie uma entrada em que a heurística do mais valioso seja muito melhor que a do mais leve. Coloque no relatório as saídas de cada programa.

!!! question
    Crie uma entrada em que a heurística do mais leve seja muito melhor que a do mais valioso. Coloque no relatório as saídas de cada programa.

!!! question
    Com base nas suas respostas acima, em quais situações a heurística do mais valioso é melhor?

!!! question
    Com base nas suas respostas acima, em quais situações a heurística do mais leve é melhor?

!!! question
    Qual a complexidade computacional das abordagens?

!!! question
    Quando uma é melhor que a outra?

!!! question
    Alguma consegue obter o melhor valor possível?



**Entrega do relatório para 17/09 as 23h59**