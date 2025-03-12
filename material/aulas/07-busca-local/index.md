# Busca local e Hill Climb

Nesta aula trabalharemos com um algoritmo chamado "Busca local", que consiste basicamente em fazer pequenas atualizações que melhoram sucessivamente uma solução.

## Busca local

Vamos implementar uma busca local para a Mochila seguindo os três algoritmos vistos na aula.

### Mochila cheia

!!! tip
    você pode utilizar o código embaralhado da [da aula 07](../06-aleatorizacao/index.md)

Para implementar a *Mochila cheia* iremos adotar a seguinte estratégia:

1. Gere uma solução aleatória.
2. Percorra novamente todos os objetos (na ordem da entrada).
3. Se um objeto couber na mochila, inclua-o.

### Substituição de objeto

Para implementar a *Substituição de objeto* iremos adotar a seguinte estratégia:

1. Gere uma solução aleatória.
2. Execute *Mochila Cheia*.
3. Para cada objeto (em ordem da entrada):
   1. Verifique, para cada objeto não usado, se uma troca aumentaria o valor da mochila.
   2. Em caso positivo, faça a troca e volte para o início do passo 2.
4. Repita enquanto for possível.

### Hill Climbing para a Mochila

Para implementar o *Hill Climbing* na Mochila, seguimos o método abaixo:

1. Para uma mochila com capacidade \( C \) e diversos itens, com seus respectivos pesos (W) e valores (V). Nosso objetivo é maximizar \( \sum V \), respeitando a restrição \( W \leq C \).

3. Podemos codificar nosso problema como uma string binária. 0 significa que o item \( i \) não foi incluído, enquanto que 1 significa que \( i \) foi incluído.
4. Supondo 10 objetos, nossa string poderia ser: `0010010000`.
5. Para este exemplo, vamos gerar 10 possíveis vizinhos a partir da modificação de um bit: `1010010000`, `0110010000`, etc.
6. Vamos computar a qualidade desses 10 vizinhos. Se houver um melhor, ele é a nova solução e repetimos o processo, até que nenhum vizinho melhor seja encontrado.


Rode os algoritmos implementados, incluindo a *Substituição de objeto* e o *Hill Climbing*, 10 vezes e retorne a melhor solução.

Houve melhoria em relação ao algoritmo embaralhado e ao probabilístico [da aula 07](../07-aleatorizacao/index.md)? Como você avalia os ganhos obtidos pelas diferentes estratégias vistas para resolver o problema da mochila?

Analise as diferenças nos tempos de execução e na qualidade das soluções nas diferentes estratégias vistas para resolver o problema da mochila.

Utilize as entradas [Entrada_1.txt](itens/Entrada_1.txt), [Entrada_2.txt](itens/Entrada_2.txt) e [Entrada_3.txt](itens/Entrada_3.txt)



**Disponibilize no seu repositório o código e as análises até as 23h59 de 14/03. *