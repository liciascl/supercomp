# 07 - Heurísticas Aleatórias para o Problema da Mochila

Nesta aula, vamos ajustar o algoritmo de busca exaustiva implementado na aula anterior para lidar com conjuntos de dados maiores e testar novas heurísticas baseadas em seleção aleatória. O objetivo é comparar a eficiência e a qualidade das soluções geradas pelos diferentes métodos.

### Refinando a Busca Exaustiva

O algoritmo de busca exaustiva da aula passada tinha a missão de encontrar a mochila mais valiosa possível considerando apenas a restrição de peso da mochila. Ou seja, o algoritmo seleciona a combinação de itens que maximiza o valor total, independentemente do peso do item, até que chegue na capacidade máxima da mochila. Vamos deixar o algoritmo mais interessante...
   
**Tarefa 1:** Faça ajustes no algoritmo para encontrar a mochila mais valiosa possível, levando em consideração o peso do item, de forma que a mochila seja a mais leve possível entre as combinações que proporcionam o valor máximo.


**Teste com Novas Entradas:**
As mochilas da atividade passada eram bem basiconas, vamos testar o código com mochilas computacionalmente mais desafiadoras:

![MUHAHAHA](itens/devil-laugh.gif)

[Entrada_1.txt](itens/Entrada_1.txt), [Entrada_2.txt](itens/Entrada_2.txt), [Entrada_3.txt](itens/Entrada_3.txt), [Entrada_4.txt](itens/Entrada_4.txt)

Meça e registre o tempo necessário para cada solução:

- Mochila com o maior valor possível atingindo o limite da capacidade de peso.
    
- Mochila com o maior valor possível e o menor peso total.

!!! Tip
    A busca exaustiva vai demorar horrores, você submeter o código no cluster e pegar o resultado um dia, quando a busca terminar <3


### Implementação de Heurísticas Aleatórias

!!! Tip
    documentação da biblioteca random [neste link](http://cplusplus.com/reference/random/)

Implemente duas novas heurísticas aleatórias para selecionar os itens da mochila:

### Tarefa 2: Embaralhar e Preencher a Mochila:

**1.** Embaralhe o vetor de itens (peso e valor).

**2.** Percorra o vetor do primeiro ao último item e, se o item couber na mochila, coloca ele lá sem pensar.

**3.** Continue até que a mochila esteja cheia ou até que os itens acabem.

Execute a heurística e compare o resultado obtido e o tempo de execução com o método de busca exaustiva.

### Tarefa 3: Seleção Aleatória Baseada em Probabilidade:

**1.** Percorra os itens sem alterar a ordem.

**2.** Para cada item, gere um número aleatório de probabilidade (entre 0 e 1).

**3.** Se a probabilidade for maior que um valor limiar definido (por exemplo, 0.5), adicione o item à mochila se ele couber; caso contrário, ignore-o, mesmo que caiba.


### Atividade 07 - Heurísticas Aleatórias par ao Problema da Mochila

   - Execute cada algorítimo aleatório 5 vezes para cada entrada ([Entrada_1.txt](itens/Entrada_1.txt), [Entrada_2.txt](itens/Entrada_2.txt), [Entrada_3.txt](itens/Entrada_3.txt), [Entrada_4.txt](itens/Entrada_4.txt)).

   - Compare o desempenho das heurísticas aleatórias com os algoritmos de busca exaustiva.

   - Analise as diferenças nos tempos de execução e na qualidade das soluções encontradas (valores obtidos e pesos das mochilas).

**Submeta no BlackBoard o link do GitHub contendo os códigos e um relatório com suas análises até as 23h59 de 02/09**
