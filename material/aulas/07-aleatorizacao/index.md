# 07 - Heurísticas Aleatórias para o Problema da Mochila

Nesta aula, vamos experimentar um conjuntos de dados maiores e testar novas heurísticas baseadas em seleção aleatória. O objetivo é comparar a eficiência e a qualidade das soluções geradas pelos diferentes métodos.

### Implementação de Heurísticas Aleatórias

!!! Tip
    documentação da biblioteca random [neste link](http://cplusplus.com/reference/random/)

Implemente duas novas heurísticas aleatórias para selecionar os itens da mochila:

### Tarefa 1: Embaralhar e Preencher a Mochila:

**1.** Embaralhe o vetor de itens (peso e valor).

**2.** Percorra o vetor do primeiro ao último item e, se o item couber na mochila, coloca ele lá sem pensar.

**3.** Continue até que a mochila esteja cheia ou até que os itens acabem.

Execute a heurística e compare o resultado obtido e o tempo de execução com o método de busca exaustiva.

### Tarefa 2: Seleção Aleatória Baseada em Probabilidade:

**1.** Percorra os itens sem alterar a ordem.

**2.** Para cada item, gere um número aleatório de probabilidade (entre 0 e 1).

**3.** Se a probabilidade for maior que um valor limiar definido (por exemplo, 0.5), adicione o item à mochila se ele couber; caso contrário, ignore-o, mesmo que caiba.


### Atividade 07 - Heurísticas Aleatórias par ao Problema da Mochila

**Teste com Novas Entradas:**
As mochilas da atividade passada eram bem basiconas, vamos testar o código com mochilas computacionalmente mais desafiadoras:

![MUHAHAHA](itens/devil-laugh.gif)

[Entrada_1.txt](itens/Entrada_1.txt), [Entrada_2.txt](itens/Entrada_2.txt), [Entrada_3.txt](itens/Entrada_3.txt), [Entrada_4.txt](itens/Entrada_4.txt)

**1.** Registre o tempo necessário para o algoritmo de busca exaustiva resolver cada Entrada.

!!! Tip
    A busca exaustiva vai demorar horrores, você pode submeter o código no cluster e pegar o resultado um dia, quando a busca terminar <3

**2.** Execute o algorítimo embaralhado da Tarefa 1 pelo menos 5 vezes para cada entrada.

**3.** Execute o algorítimo probabilítico da Tarefa 2 pelo menos 5 vezes para cada entrada.

**4.** Analise as diferenças nos tempos de execução e na qualidade das soluções encontradas (valores obtidos e pesos das mochilas) das heurísticas aleatórias e da busca exaustiva.

**Submeta no BlackBoard o link do GitHub contendo os códigos e um relatório com suas análises até as 23h59 de 02/09**
