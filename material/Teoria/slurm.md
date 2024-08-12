# SLURM

### O que é SLURM?

**SLURM** (Simple Linux Utility for Resource Management) é um gerenciador de workload open-source amplamente utilizado em clusters de computação de alto desempenho (HPC). Ele é responsável por alocar recursos de computação (como CPUs, memória e GPUs) aos usuários e suas tarefas, gerenciar filas de jobs, monitorar o uso de recursos e agendar a execução de tarefas de forma eficiente.

### Principais Funcionalidades do SLURM

1. **Alocação de Recursos**: SLURM distribui recursos de computação, como nós e processadores, conforme solicitado pelos usuários. Ele assegura que os recursos são utilizados de maneira eficiente e equitativa.
2. **Submissão de Jobs**: Usuários podem submeter jobs (tarefas de computação) ao SLURM, que coloca esses jobs em uma fila e os executa quando os recursos necessários estão disponíveis.
3. **Monitoramento e Gerenciamento**: SLURM monitoriza o estado dos jobs, nós e partições do cluster, fornecendo ferramentas para verificar o status e a utilização de recursos.
4. **Políticas de Priorização**: Implementa políticas para priorizar jobs com base em vários fatores, como tempo de espera, utilização de recursos e prioridades definidas pelo administrador do cluster.

### Comandos Principais do SLURM

Alguns dos comandos principais usados no SLURM:

1. **sbatch**: É um comando do SLURM usado para submeter scripts de jobs para execução em um cluster. Esses scripts contêm instruções sobre como os recursos devem ser alocados e quais comandos devem ser executados
2. **scancel**: Cancela um job pendente ou em execução.
3. **scontrol**: Ferramenta administrativa usada para visualizar e/ou modificar o estado do SLURM. Muitos comandos `scontrol` só podem ser executados pelo adminstrador do sistema.
4. **sinfo**: Relata o estado das filas e nós gerenciados pelo SLURM, com várias opções de filtragem, ordenação e formatação.
5. **sprio**: Exibe uma visão detalhada dos componentes que afetam a prioridade de um job.
6. **squeue**: Relata o estado dos jobs em execução em ordem de prioridade e depois os jobs pendentes em ordem de prioridade.
7. **srun**: Submete um job para execução e faz o pedido de alocação dos recursos da maquina.
8. **sstat**: Obtém informações sobre os recursos utilizados por um job em execução.
9. **strigger**: Define, obtém ou visualiza gatilhos de eventos, como nós caindo ou jobs se aproximando do limite de tempo.
10. **sview**: Interface gráfica para obter e atualizar informações de estado para jobs, partições e nós gerenciados pelo SLURM.

### Exemplos de Uso para Cada Comando SLURM

### 1. **sbatch**

**Descrição:** Submete scripts de jobs para execução em um cluster. Esses scripts contêm instruções sobre como os recursos devem ser alocados e quais comandos devem ser executados.

**Exemplo de Script:**

```bash
#!/bin/bash
#SBATCH --job-name=meu_job        # Nome do job
#SBATCH --output=meu_job.out      # Arquivo de saída
#SBATCH --error=meu_job.err       # Arquivo de erro
#SBATCH --ntasks=1                # Número de tarefas
#SBATCH --cpus-per-task=4         # Número de CPUs por tarefa
#SBATCH --mem=4G                  # Memória total alocada para o job
#SBATCH --time=00:02:00           # Tempo máximo de execução (hh:mm:ss)
#SBATCH --partition=normal        # Fila do cluster

echo "Iniciando o job"
sleep 60
echo "Job finalizado"

```

**Explicação das Flags:**

- **`-job-name=meu_job`**: Define o nome do job como `meu_job`.
- **`-output=meu_job.out`**: Especifica o arquivo onde a saída padrão do job será registrada.
- **`-error=meu_job.err`**: Especifica o arquivo onde os erros serão registrados.
- **`-ntasks=1`**: Define o número de tarefas (processos) a serem utilizados pelo job.
- **`-cpus-per-task=4`**: Aloca 4 CPUs para cada tarefa.
- **`-mem=4G`**: Especifica que 4 GB de memória serão alocados para o job.
- **`-time=00:02:00`**: Define o tempo máximo de execução do job como 2 minutos.
- **`-partition=normal`**: Especifica a fila do cluster onde o job será executado.

### 2. **scancel**

**Descrição:** Cancela um job pendente ou em execução.

**Exemplo:**

```bash
scancel 12345

```

**Explicação das Flags:**

- **`12345`**: Especifica o ID do job que deve ser cancelado.

### 3. **scontrol**

**Descrição:** Ferramenta administrativa usada para visualizar e/ou modificar o estado do SLURM. Muitos comandos `scontrol` só podem ser executados pelo administrador do sistema.

**Exemplo:**

```bash
scontrol show job 12345

```

**Explicação das Flags:**

- **`show job 12345`**: Exibe informações detalhadas sobre o job com ID `12345`.

### 4. **sinfo**

**Descrição:** Relata o estado das filas e nós gerenciados pelo SLURM.

**Exemplo:**

```bash
sinfo

```

### 5. **sprio**

**Descrição:** Exibe uma visão detalhada dos componentes que afetam a prioridade de um job.

**Exemplo:**

```bash
sprio

```

### 6. **squeue**

**Descrição:** Relata o estado dos jobs em execução em ordem de prioridade e depois os jobs pendentes em ordem de prioridade.

**Exemplo:**

```bash
squeue -u username

```

**Explicação das Flags:**

- **`u username`**: Filtra a saída para mostrar apenas os jobs do usuário `username`.

### 7. **srun**

**Descrição:** Submete um job para execução e faz o pedido de alocação dos recursos da máquina.

**Exemplo:**

```bash
srun -N 1 -n 1 --time=00:01:00 ./meu_programa

```

**Explicação das Flags:**

- **`N 1`**: Especifica que 1 nó deve ser alocado.
- **`n 1`**: Define que 1 tarefa deve ser executadas.
- **`-time=00:01:00`**: Define o tempo máximo de execução do job como 1 minuto.
- **`./meu_programa`**: Especifica o programa a ser executado.

### 8. **sstat**

**Descrição:** Obtém informações sobre os recursos utilizados por um job em execução.

**Exemplo:**

```bash
sstat -j 12345 --format=JobID,MaxRSS,AveCPU

```

**Explicação das Flags:**

- **`j 12345`**: Especifica o ID do job para o qual queremos informações.
- **`-format=JobID,MaxRSS,AveCPU`**: Define os campos a serem exibidos na saída:
    - **JobID**: ID do job.
    - **MaxRSS**: Máximo de memória física utilizada.
    - **AveCPU**: Tempo médio de CPU utilizado.

### 9. **strigger**

**Descrição:** Define, obtém ou visualiza gatilhos de eventos, como nós caindo ou jobs se aproximando do limite de tempo.

**Exemplo:**
Vamos supor que você queira configurar um gatilho (`strigger`) para monitorar o uso de memória de um job específico e enviar um alerta quando o uso de memória ultrapassar um certo limite. Aqui está um exemplo completo:

```bash
strigger --set --jobid=<JOB_ID> --threshold=500 --action="echo 'Atenção: O uso de memória ultrapassou 500 MB!'"
```
### Explicação do Comando:

- **`strigger`**: Comando para configurar um gatilho no SLURM.
- **`--set`**: Indica que estamos criando um novo gatilho.
- **`--jobid=<JOB_ID>`**: Especifica o ID do job que queremos monitorar. Substitua `<JOB_ID>` pelo ID real do job.
- **`--threshold=500`**: Define o limiar de 500 MB de uso de memória. Quando o job usar mais de 500 MB, o gatilho será ativado.
- **`--action="echo 'Atenção: O uso de memória ultrapassou 500 MB!'"`**: Define a ação que será executada quando o gatilho for ativado. Neste caso, a ação é um simples comando `echo` que imprime uma mensagem de alerta.

Este gatilho será acionado quando o job especificado pelo `<JOB_ID>` ultrapassar 500 MB de uso de memória. A ação definida (`echo`) será executada, e você verá a mensagem "Atenção: O uso de memória ultrapassou 500 MB!" no terminal ou no arquivo de saída do job.

Você pode personalizar o comando `strigger` para outras situações, como monitorar o tempo restante de um job, detectar falhas de nós, ou monitorar o uso de CPU, simplesmente ajustando os parâmetros e as ações conforme necessário.

### Tipos Comuns de Eventos Monitoráveis pelo `strigger`:

1. **Tempo Restante (`TIME_LIMIT`)**:
   - **Descrição**: Aciona o gatilho quando um job se aproxima de seu limite de tempo de execução.
   - **Uso do `--threshold`**: Especifica o tempo restante em segundos.
   - **Exemplo**: `strigger --set --jobid=<JOB_ID> --threshold=60 --action="echo 'Job está a 60 segundos do limite de tempo!'"`
  
2. **Uso de Memória (`MEMORY`)**:
   - **Descrição**: Aciona o gatilho quando o uso de memória de um job ultrapassa um certo limite.
   - **Uso do `--threshold`**: Especifica a quantidade de memória usada, geralmente em megabytes (MB).
   - **Exemplo**: `strigger --set --jobid=<JOB_ID> --threshold=1024 --action="echo 'Job ultrapassou 1GB de memória!'"`
  
3. **Falha de Nó (`NODE_FAIL`)**:
   - **Descrição**: Aciona o gatilho quando um nó falha ou fica indisponível.
   - **Uso do `--threshold`**: Não aplicável diretamente. O gatilho é acionado quando o evento de falha ocorre.
   - **Exemplo**: `strigger --set --node=<NODE_NAME> --event=NODE_FAIL --action="echo 'Nó falhou!'"`
  
4. **Falha de Job (`JOB_FAIL`)**:
   - **Descrição**: Aciona o gatilho quando um job falha por qualquer motivo.
   - **Uso do `--threshold`**: Não aplicável diretamente. O gatilho é acionado quando o evento de falha ocorre.
   - **Exemplo**: `strigger --set --jobid=<JOB_ID> --event=JOB_FAIL --action="echo 'Job falhou!'"`
  
5. **Início de Job (`JOB_START`)**:
   - **Descrição**: Aciona o gatilho quando um job começa a ser executado.
   - **Uso do `--threshold`**: Não aplicável diretamente. O gatilho é acionado quando o evento de início ocorre.
   - **Exemplo**: `strigger --set --jobid=<JOB_ID> --event=JOB_START --action="echo 'Job iniciou!'"`
  
6. **Finalização de Job (`JOB_END`)**:
   - **Descrição**: Aciona o gatilho quando um job termina sua execução, independentemente de ter sido concluído com sucesso ou não.
   - **Uso do `--threshold`**: Não aplicável diretamente. O gatilho é acionado quando o evento de finalização ocorre.
   - **Exemplo**: `strigger --set --jobid=<JOB_ID> --event=JOB_END --action="echo 'Job terminou!'"`
  
7. **Limite de CPU (`CPU_LIMIT`)**:
   - **Descrição**: Aciona o gatilho quando o uso de CPU de um job ultrapassa um certo limite.
   - **Uso do `--threshold`**: Especifica o uso de CPU em segundos de CPU ou em porcentagem.
   - **Exemplo**: `strigger --set --jobid=<JOB_ID> --threshold=80 --action="echo 'Job ultrapassou 80% do uso de CPU!'"`
  
8. **Submissão de Job (`JOB_SUBMIT`)**:
   - **Descrição**: Aciona o gatilho quando um job é submetido para execução.
   - **Uso do `--threshold`**: Não aplicável diretamente. O gatilho é acionado quando o evento de submissão ocorre.
   - **Exemplo**: `strigger --set --user=<USER_NAME> --event=JOB_SUBMIT --action="echo 'Um job foi submetido!'"`
  
9. **Queda de Partição (`PARTITION_DOWN`)**:
   - **Descrição**: Aciona o gatilho quando uma partição inteira do cluster cai ou fica indisponível.
   - **Uso do `--threshold`**: Não aplicável diretamente. O gatilho é acionado quando o evento de queda da partição ocorre.
   - **Exemplo**: `strigger --set --partition=<PARTITION_NAME> --event=PARTITION_DOWN --action="echo 'Partição caiu!'"`
  
10. **Chegada de Job ao Limiar de Tempo (`TIME_LIM_REACHED`)**:
    - **Descrição**: Similar ao `TIME_LIMIT`, mas pode ser mais específico para quando o tempo limite é alcançado, não apenas quando está próximo.
    - **Uso do `--threshold`**: Especifica o tempo restante ou o evento do tempo limite.
    - **Exemplo**: `strigger --set --jobid=<JOB_ID> --event=TIME_LIM_REACHED --action="echo 'Tempo limite alcançado!'"`


### Exemplo 1: Gatilho de Tempo Restante

**Objetivo:** Enviar uma notificação quando um job estiver a 120 segundos de atingir seu limite de tempo.

```bash
strigger --set --jobid=<JOB_ID> --threshold=120 --action="echo 'Aviso: Job está a 120 segundos do limite de tempo!'"
```

- **`--jobid=<JOB_ID>`**: Substitua `<JOB_ID>` pelo ID real do job.
- **`--threshold=120`**: O gatilho será acionado quando o job estiver a 120 segundos do limite de tempo.
- **`--action="echo 'Aviso: Job está a 120 segundos do limite de tempo!'"`**: A ação que será executada é imprimir uma mensagem de aviso.

### Exemplo 2: Gatilho de Falha de Job

**Objetivo:** Notificar quando um job falhar.

```bash
strigger --set --jobid=<JOB_ID> --event=JOB_FAIL --action="echo 'Alerta: Job falhou!'"
```

- **`--event=JOB_FAIL`**: Configura o gatilho para disparar quando o evento de falha do job ocorrer.
- **`--action="echo 'Alerta: Job falhou!'"`**: A ação que será executada é imprimir uma mensagem de alerta.

### Exemplo 3: Gatilho de Uso de CPU

**Objetivo:** Enviar uma notificação quando o uso de CPU de um job ultrapassar 80%.

```bash
strigger --set --jobid=<JOB_ID> --threshold=80 --action="echo 'Aviso: Uso de CPU ultrapassou 80%!'"
```

- **`--threshold=80`**: Define o limiar de 80% de uso de CPU. O gatilho será acionado quando o uso de CPU ultrapassar este valor.
- **`--action="echo 'Aviso: Uso de CPU ultrapassou 80%!'"`**: A ação que será executada é imprimir uma mensagem de aviso.

### Exemplo 4: Gatilho de Início de Job

**Objetivo:** Notificar quando um job começar a ser executado.

```bash
strigger --set --jobid=<JOB_ID> --event=JOB_START --action="echo 'Informação: Job iniciou!'"
```

- **`--event=JOB_START`**: Configura o gatilho para disparar quando o evento de início do job ocorrer.
- **`--action="echo 'Informação: Job iniciou!'"`**: A ação que será executada é imprimir uma mensagem de informação.

### Exemplo 5: Gatilho de Finalização de Job

**Objetivo:** Notificar quando um job terminar sua execução.

```bash
strigger --set --jobid=<JOB_ID> --event=JOB_END --action="echo 'Informação: Job terminou!'"
```

- **`--event=JOB_END`**: Configura o gatilho para disparar quando o evento de finalização do job ocorrer.
- **`--action="echo 'Informação: Job terminou!'"`**: A ação que será executada é imprimir uma mensagem de informação.

### Exemplo 6: Gatilho de Queda de Partição

**Objetivo:** Notificar quando uma partição cair.

```bash
strigger --set --partition=<PARTITION_NAME> --event=PARTITION_DOWN --action="echo 'Alerta: Partição <PARTITION_NAME> caiu!'"
```

- **`--partition=<PARTITION_NAME>`**: Substitua `<PARTITION_NAME>` pelo nome real da partição.
- **`--event=PARTITION_DOWN`**: Configura o gatilho para disparar quando a partição ficar indisponível.
- **`--action="echo 'Alerta: Partição <PARTITION_NAME> caiu!'"`**: A ação que será executada é imprimir uma mensagem de alerta.

### Exemplo 7: Gatilho de Submissão de Job

**Objetivo:** Notificar quando um job for submetido.

```bash
strigger --set --user=<USER_NAME> --event=JOB_SUBMIT --action="echo 'Informação: Um job foi submetido pelo usuário <USER_NAME>!'"
```

- **`--user=<USER_NAME>`**: Substitua `<USER_NAME>` pelo nome real do usuário.
- **`--event=JOB_SUBMIT`**: Configura o gatilho para disparar quando um job for submetido.
- **`--action="echo 'Informação: Um job foi submetido pelo usuário <USER_NAME>!'"`**: A ação que será executada é imprimir uma mensagem de informação.

### Exemplo 8: Gatilho de Falha de Nó

**Objetivo:** Notificar quando um nó falhar.

```bash
strigger --set --event=NODE_FAIL --node=<NODE_NAME> --action="echo 'Alerta: O nó <NODE_NAME> falhou!'"
```

- **`--node=<NODE_NAME>`**: Substitua `<NODE_NAME>` pelo nome real do nó.
- **`--event=NODE_FAIL`**: Configura o gatilho para disparar quando o nó falhar.
- **`--action="echo 'Alerta: O nó <NODE_NAME> falhou!'"`**: A ação que será executada é imprimir uma mensagem de alerta.

### Exemplo 9: Gatilho de Uso de Memória

**Objetivo:** Enviar uma notificação quando o uso de memória de um job ultrapassar 1 GB.

```bash
strigger --set --jobid=<JOB_ID> --threshold=1024 --action="echo 'Aviso: O uso de memória ultrapassou 1 GB!'"
```

- **`--threshold=1024`**: Define o limiar de 1024 MB (1 GB) de uso de memória.
- **`--action="echo 'Aviso: O uso de memória ultrapassou 1 GB!'"`**: A ação que será executada é imprimir uma mensagem de aviso.

Esses exemplos mostram como configurar gatilhos (`strigger`) para monitorar diferentes eventos no SLURM e executar ações específicas quando esses eventos ocorrem. Você pode ajustar os parâmetros de acordo com suas necessidades específicas


### 10. **sview**

**Descrição:** Interface gráfica para obter e atualizar informações de estado para jobs, partições e nós gerenciados pelo SLURM.

**Exemplo:**

```bash
sview

```

**Explicação das Flags:**

- **`sview`**: Inicia a interface gráfica do SLURM para monitoramento e gerenciamento de recursos.