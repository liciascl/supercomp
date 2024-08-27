# Aula 06: Problema da Mochila com Busca Exaustiva

O problema da mochila é um dos problemas clássicos da computação e é amplamente estudado em otimização de algoritmos e computação. 

Imagine que você está se preparando para uma viagem e tem uma mochila com capacidade limitada de peso. Você tem vários itens à sua disposição, cada um com um peso específico e um valor associado. Seu objetivo é escolher quais itens colocar na mochila para maximizar o valor total, sem ultrapassar a capacidade máxima de peso que a mochila pode carregar.

Suponha que você tenha os seguintes itens:

**Item 1:** Peso = 2 kg, Valor = $3$

**Item 2:** Peso = 3 kg, Valor = $4$

**Item 3:** Peso = 4 kg, Valor = $5$

**Item 4:** Peso = 5 kg, Valor = $8$

E sua mochila pode carregar até **8 kg**. A questão é: quais itens você deve escolher para maximizar o valor total na mochila, sem ultrapassar a capacidade de 8 kg?

Se você tentar todas as combinações possíveis de itens, verá que algumas opções permitem carregar mais valor do que outras. Por exemplo:

Se você escolher os itens 1 e 4, o peso total será 7 kg e o valor total será $11$.

Se escolher os itens 2 e 3, o peso total será 7 kg e o valor total será $9$.

Neste caso, a melhor combinação seria escolher os itens 1 e 4, que oferecem o maior valor total de $11$ sem exceder o peso máximo de 8 kg.

#### Desafios do Problema:

O problema da mochila torna-se cada vez mais complexo à medida que o número de itens aumenta. Com mais itens, o número de combinações possíveis cresce exponencialmente, tornando a busca pela solução ótima um desafio computacional significativo. Por isso, algoritmos eficientes são necessários para resolver o problema em casos de grande escala.

A atividade desta aula consiste em implementar heurísticas para a solução do problema da **Mochila binária**. 


Abaixo você encontrará as listas contendo peso e valor dos itens para organizar na mochila.

[Entrada 1](itens/in1.txt), [Entrada 2](itens/in2.txt), [Entrada 3](itens/in3.txt), [Entrada 4](itens/in4.txt) 

Neste exercício, você implementará uma solução para o problema da mochila usando a técnica de busca exaustiva em C++. O objetivo é maximizar o valor dos objetos dentro da mochila sem exceder sua capacidade. 

**Descrição do Problema:**  
Dado um conjunto de N objetos, cada um com um peso \( w_i \) e um valor \( v_i \), selecione uma combinação de objetos que maximize o valor total sem ultrapassar a capacidade W da mochila.

- **Entrada:**
  ```
  N W
  w1 v1
  w2 v2
  ...
  wN vN
  ```

  Onde:
**N:** Número de objetos

**W:** Capacidade da mochila 

**Wi:** Peso do objeto

**Vi:** Valor do objeto.

Para resolver o problema da mochila usando busca exaustiva, precisamos avaliar todas as combinações possíveis de itens. Uma abordagem comum é usar recursão para explorar todas as possibilidades.

#### O que é Recursão?

Recursão é uma técnica fundamental na programação, onde uma função chama a si mesma para resolver subproblemas menores de uma tarefa maior. A ideia central é dividir um problema complexo em partes menores, que podem ser resolvidas de forma mais simples. Em um algoritmo recursivo, o problema é resolvido ao fazer chamadas sucessivas para a mesma função, mas com parâmetros ajustados que aproximam a solução a cada passo.

**Exemplificando...**

Considere a tarefa de calcular o fatorial de um número \( n \). O fatorial de \( n \), representado como \( n! \), é o produto de todos os números inteiros de 1 até \( n \). Podemos definir isso de forma recursiva:
- \( 0! = 1 \) (caso base)
- \( n! = n \times (n-1)! \) (caso recursivo)

Usando recursão, a função que calcula o fatorial chama a si mesma com o valor \( n-1 \), até que atinge o caso base.

#### Recursão no Problema da Mochila

No contexto do problema da mochila, a recursão é utilizada para explorar todas as combinações possíveis de itens que podem ser colocados na mochila. A ideia é considerar, para cada item, duas possibilidades: incluí-lo na mochila ou não incluí-lo. Para cada uma dessas escolhas, o problema é reduzido ao mesmo problema, mas com um item a menos e, no caso de inclusão, com uma capacidade reduzida na mochila.

A estrutura recursiva para resolver o problema da mochila pode ser descrita da seguinte forma:

1. **Caso Base:** Se não há mais itens a serem considerados ou se a capacidade 
restante da mochila é zero, o valor total é zero.

2. **Caso Recursivo:** Para cada item \( i \):

Se incluir o item \( i \): resolva o problema para os itens restantes com a capacidade reduzida.

Se não incluir o item \( i \): resolva o problema para os itens restantes sem alterar a capacidade.

Retorne a melhor das duas soluções.

### Atividade 06: Problema da Mochila com Busca Exaustiva

**Objetivo:**
Implemente uma solução para o problema da mochila usando a técnica de busca exaustiva através de recursão em C++. O objetivo é maximizar o valor dos objetos dentro da mochila sem exceder sua capacidade.

Utilize os arquivos de entrada [Entrada 1](itens/in1.txt), [Entrada 2](itens/in2.txt), [Entrada 3](itens/in3.txt) e [Entrada 4](itens/in4.txt) para validar sua implementação. Verifique se o programa consegue determinar a combinação correta para maximizar o valor na mochila para cada arquivo de entrada.

Compare o tempo de execução para os diferentes arquivos de entrada. Considerando que o algoritmo explora todas as combinações possíveis, discuta suas limitações em termos de tempo de execução e uso de memória.

Submeta no BlackBoard o link do repositório com o código e as análises até as 23h59 de 30/08. 