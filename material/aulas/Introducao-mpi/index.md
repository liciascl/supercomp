# Introdução a MPI

Message Passing Interface (MPI) é um padrão para comunicação de dados em computação paralela. Existem várias modalidades de computação paralela, e dependendo do problema que se está tentando resolver, pode ser necessário passar informações entre os vários processadores ou nós de um cluster, e o MPI oferece uma infraestrutura para essa tarefa.

Para iniciarmos o nosso estudo de MPI, implemente os desafios abaixo, entendendo como encadear `sends` e `receives`, e o impacto nos resultados.

## Exercícios MPI

### Exercício 1: Comunicação entre Dois Processos
    **Descrição:**  
    Escreva um programa MPI em que o processo de rank 0 envia uma mensagem `"Olá"` para o processo de rank 1, e o processo 1 responde com `"Oi"` ao processo 0.

    **Instruções:**
    1. O processo com rank 0 envia uma mensagem `"Olá"` para o processo com rank 1.
    2. O processo com rank 1 responde com `"Oi"` para o processo com rank 0.
    3. Exiba as mensagens trocadas no terminal.

### Exercício 2: Anel de Processos
    **Descrição:**  
    Crie um programa MPI onde cada processo envia uma mensagem para o próximo processo, formando um anel. O último processo envia a mensagem de volta ao primeiro.

    **Instruções:**
    1. Cada processo com rank `i` envia uma mensagem para o processo de rank `i+1`.
    2. O último processo (com o maior rank) envia de volta ao processo com rank 0.
    3. O número de processos deve ser maior que 2.
    4. Exiba a mensagem recebida por cada processo no terminal.

    **Dicas:**
    - O processo com rank 0 envia e recebe por último para fechar o anel.

### Exercício 3: Difusão Linear
    **Descrição:**  
    O processo com rank 0 inicia enviando uma mensagem para o processo 1. O processo 1, ao receber a mensagem, envia-a para o processo 2, e assim sucessivamente, até que todos os processos tenham recebido a mensagem.

    **Instruções:**
    1. O processo com rank 0 envia uma mensagem inicial para o processo com rank 1.
    2. Cada processo recebe a mensagem do processo anterior e a envia para o próximo.
    3. O último processo exibe a mensagem no terminal.

    **Dicas:**
    - Cada processo (exceto o último) deve executar um `MPI_Recv` seguido de um `MPI_Send`.

### Exercício 4: Comunicação Múltipla
    **Descrição:**
    Implemente um programa MPI onde o processo com rank 0 envia diferentes mensagens para cada um dos outros processos. Cada processo (exceto o 0) recebe sua mensagem e imprime o conteúdo.
    
    **Instruções:**
    1. O processo com rank 0 envia mensagens exclusivas para cada processo, incluindo o número do processo na mensagem (ex: "Mensagem para o processo X").
    2. Cada processo, ao receber sua mensagem, imprime no terminal o que recebeu.
    3. Use um laço no processo 0 para enviar as mensagens para todos os outros.
    
    **Dicas:**
    - Lembre-se de utilizar tags nas mensagens para diferenciá-las, se necessário.

### Exercício 5: Somas Paralelas
    **Descrição:**
    Crie um programa MPI em que cada processo recebe um valor numérico exclusivo, soma-o ao valor de um processo anterior, e o último processo exibe a soma total de todos os processos.

    **Instruções:**
    1. O processo 0 inicia com um valor inicial e o envia para o processo 1.
    2. Cada processo soma seu valor ao que recebeu do processo anterior e envia a soma para o próximo processo.
    3. O último processo imprime a soma total.
    
    **Dicas:**
    - Cada processo realiza uma soma antes de enviar o resultado ao próximo.


### Exercício 6: Alternância de Mensagens
    **Descrição:** 
    Escreva um programa em que o processo 0 envia uma mensagem para o processo 1, o qual envia uma mensagem de volta para o processo 0. O processo 0, então, envia uma segunda mensagem para o processo 2, e assim sucessivamente até que todos os processos tenham sido envolvidos na comunicação.
    
    **Instruções:**
    1. O processo com rank 0 envia uma mensagem para o processo 1.
    2. O processo 1 responde ao processo 0.
    3. O processo 0 então envia uma segunda mensagem para o processo 2, que responde.
    4. Continue este padrão até o último processo.

### Entrega

Para cada exercício, entregue:
- Seu código 
- O arquivo `.slurm`
- O output do seu resultado


**Faça a sua subimissão até as 23h59 de 05/05** 