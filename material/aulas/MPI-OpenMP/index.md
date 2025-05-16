# MPI e OpenMP

Agora que vocês aprenderam sobre **MPI** e já trabalharam bastante com **OpenMP**, vamos juntar o melhor dos dois mundos!

Combinar essas duas abordagens é uma estratégia poderosa em computação de alto desempenho. Enquanto o **OpenMP** permite paralelizar tarefas de forma simples e eficiente dentro de uma mesma máquina (memória compartilhada), o **MPI** possibilita distribuir o trabalho entre várias máquinas (memória distribuída).

Ao usar **MPI + OpenMP juntos**, conseguimos:

- **Aproveitar ao máximo os recursos de clusters de alto desempenho**, que possuem múltiplos nós (com MPI) e múltiplos núcleos por nó (com OpenMP).
- **Reduzir o overhead de comunicação**, evitando o uso excessivo de mensagens MPI quando podemos usar threads locais com OpenMP.
- **Escalar aplicações de forma mais eficiente**, especialmente em tarefas intensivas como simulações científicas, análise de dados em larga escala e modelagem computacional.


# Exercícios MPI + OpenMP

A seguir, você encontrará uma série de exercícios que combinam MPI e OpenMP para explorar os conceitos de paralelismo distribuído e local.


Os Slides estão [disponíveis aqui](slides.pdf)

---

## Exercício 1: Cálculo do Quadrado de cada Elemento da Matriz

**Objetivo:** Compreender os efeitos da paralelização com MPI e OpenMP.

**Descrição:**

1. Volte ao exercício do slide e execute com tamanhos diferentes da matriz (cada vez maiores) e observe o tempo necessário para rodar:
   - a. Mantendo a paralelização local com OpenMP;
   - b. Removendo a paralelização local com OpenMP;

---

## Exercício 2: Cálculo do Quadrado de cada Elemento da Matriz

**Objetivo:** Compreender os efeitos colaterais da paralelização com MPI.

**Descrição:**

1. Volte ao exercício do slide e execute com diferentes quantidades de máquinas.
2. Observe a divisão dos dados realizada com `MPI_Scatter`.
3. Reflita: como proteger o código contra efeitos colaterais de separações “estranhas” dos dados?

---

## Exercício 3: Cálculo Distribuído da Média de um Array

**Objetivo:** Usar MPI e OpenMP para distribuir o cálculo da média de um vetor de N valores.

**Descrição:**

1. A inicialização do vetor pode ocorrer no `rank=0` ou localmente em cada máquina.
2. Cada máquina realiza a soma de sua parte do vetor, usando OpenMP para paralelizar esse cálculo.
3. O `rank=0` recebe as somas parciais de cada processo, calcula a soma total e divide por `N` para obter a média.

---

## Exercício 4: Busca em Vetor Distribuído

**Objetivo:** Implementar uma busca em paralelo para encontrar um valor em um vetor distribuído. Divida o vetor entre os nós MPI e use OpenMP para realizar a busca em cada parte do vetor. Retorne todas as posições encontradas.

**Descrição:**

1. Distribuir o vetor entre os processos MPI.
2. Utilizar OpenMP para realizar a busca paralela em cada subvetor.
3. Retornar todas as posições encontradas do valor buscado.
