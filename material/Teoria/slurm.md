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

```bash
strigger --set --jobid=12345 --signal=SIGKILL --time=60

```

**Explicação das Flags:**

- **`-set`**: Define um novo gatilho.
- **`-jobid=12345`**: Especifica o ID do job para o qual o gatilho deve ser configurado.
- **`-signal=SIGKILL`**: Define o sinal a ser enviado (SIGKILL).
- **`-time=60`**: Define que o sinal deve ser enviado 60 segundos antes do término do job.

### 10. **sview**

**Descrição:** Interface gráfica para obter e atualizar informações de estado para jobs, partições e nós gerenciados pelo SLURM.

**Exemplo:**

```bash
sview

```

**Explicação das Flags:**

- **`sview`**: Inicia a interface gráfica do SLURM para monitoramento e gerenciamento de recursos.