### Aula 1: Escalando Processos de Big Data e IA com HPC no Cluster Franky

Compreender as limitações do processamento de dados em um ambiente computacional tradicional é apenas o primeiro passo. Agora, é hora de explorar como HPC pode transformar essas limitações em vantagens. Nesta atividade, você fará o pré-processamento e o treinamento do modelo de IA utilizando Dask e Apache Spark no Cluster Franky, se você não tem ideia do que essas coisas significam, fique tranquilinho(a), vamos um passo de cada vez.

O objetivo é demonstrar como técnicas de paralelismo e distribuição de carga podem melhorar drasticamente a eficiência computacional. Ao comparar os tempos de execução e a precisão do modelo entre os ambientes local e distribuído, você entenderá como o HPC pode ser utilizado para processar grandes volumes de dados de maneira rápida e eficaz.

!!! warning "Atenção!"
    A entrega da atividade só poderá ser realizada até as 23h59 do dia da respectiva aula. 
    A somatória de todas as entregas válidas atribui até **2 pontos na nota final**.
    A descrição da atividade está no final da página.


### **Parte 0: Configurando seu acesso ao Cluster Franky**

Para ter acesso ao Cluster Franky você precisa configurar suas credenciais de acesso e realizar acesso remoto via SSH.

1. **Acesse o Blackboard**: 
   - Entre no Blackboard.
   - Navegue até a seção de materiais do curso onde a pasta com o par de chaves foi disponibilizada.

2. **Baixe a Pasta**:
   - Localize a pasta nomeada com o seu usuário.
   - Faça o download da pasta completa, que contém os arquivos `id_rsa` (chave privada) e `id_rsa.pub` (chave pública).


Dependendo do sistema operacional que você utiliza, siga as instruções abaixo para configurar corretamente sua chave privada.

#### **Para Macbook ou Linux:**

1. **Mova a Chave Privada**:
   - Abra o terminal.
   - Navegue até a pasta onde a chave privada (`id_rsa`) foi baixada.
   - Mova a chave para o diretório `.ssh` em sua home:
     ```bash
     mv id_rsa ~/.ssh/
     ```

2. **Ajuste as Permissões da Chave Privada**:
   - Garanta que apenas você possa ler o arquivo:
     ```bash
     chmod 600 ~/.ssh/id_rsa
     ```

3. **Acessar o Cluster via SSH**:
   - Conecte-se ao cluster utilizando o comando SSH:
     ```bash
     ssh -i ~/.ssh/id_rsa seu_usuario_insper@ip_do_cluster
     ```
   - Substitua `seu_usuario_insper` pelo seu nome de usuário Insper e `cluster_endereco` pelo endereço de IP fornecido durante a aula.

#### **Para Windows:**

1. **Usando OpenSSH :**

   - **Mover a Chave Privada**:
     - Abra o PowerShell ou Windows Terminal.
     - Navegue até a pasta onde a chave privada (`id_rsa`) foi baixada.
     - Mova a chave para a pasta `.ssh` em seu diretório de usuário:
       ```powershell
       mkdir $env:USERPROFILE\.ssh
       mv id_rsa $env:USERPROFILE\.ssh\
       ```

   - **Ajustar Permissões da Chave Privada**:
     - Certifique-se de que as permissões estão corretas:
       ```powershell
       icacls $env:USERPROFILE\.ssh\id_rsa /inheritance:r /grant:r "$($env:USERNAME):(R)"
       ```

   - **Acessar o Cluster via SSH**:
     - Conecte-se ao cluster usando o comando:
       ```powershell
       ssh -i $env:USERPROFILE\.ssh\id_rsa seu_usuario_insper@ip_do_cluster
       ```

### **Passo 3: Configurar o VS Code para Acesso Remoto ao Cluster**

1. **Instale a Extensão Remote - SSH**:
   - Abra o VS Code.
   - Vá para a aba de extensões (ícone de quadrado no lado esquerdo).
   - Pesquise por "Remote - SSH" e instale a extensão oficial da Microsoft.

2. **Configurar o Acesso Remoto**:

   - **Adicione uma Nova Configuração SSH**:
     - Pressione `Ctrl+Shift+P` (ou `Cmd+Shift+P` no Mac) para abrir o painel de comandos.
     - Digite `Remote-SSH: Add New SSH Host...` e selecione a opção.
     - Insira o comando SSH que você utilizou anteriormente:
       ```bash
       ssh -i ~/.ssh/id_rsa seu_usuario@cluster_endereco
       ```
     - Escolha o arquivo de configuração padrão (`~/.ssh/config` para Mac/Linux ou `C:\Users\seu_usuario\.ssh\config` para Windows).

   - **Conectar ao Cluster**:
     - Pressione `Ctrl+Shift+P` (ou `Cmd+Shift+P` no Mac) novamente e digite `Remote-SSH: Connect to Host...`.
     - Selecione o host configurado no passo anterior.
     - O VS Code abrirá uma nova janela conectada ao ambiente remoto do cluster.

3. **Gerenciar Projetos Remotamente**:
   - Após a conexão, você pode abrir pastas e arquivos no cluster diretamente pelo VS Code.
   - Utilize os recursos do VS Code, como o terminal integrado e o depurador, para trabalhar eficientemente no ambiente remoto.


###  Executando a Atividade 0 no Cluster Franky usando SLURM

Neste exercício, você irá colocar em prática os conceitos de HPC aprendidos, executando o código desenvolvido na aula anterior, utilizando o SLURM para gerenciamento de recursos. O objetivo é comparar o tempo de execução e a eficiência computacional entre o ambiente local e o ambiente de cluster.


#### **1. Preparar o Código para Execução no Cluster**

1. **Verifique o Código da Atividade 0:**
   - Certifique-se de que o código da Atividade 0 está funcionando corretamente no seu ambiente local.
   - O código deve estar preparado para ser executado sem intervenção manual.

2. **Criar um Script SLURM para Submeter o Job:**

   Crie um arquivo de script chamado `mnist_slurm_job.sh`, que será usado para submeter o seu código no cluster Franky através do SLURM.

   ```bash
   #!/bin/bash
   # Nome do Job
   #SBATCH --job-name=mnist_hpc
   # Quantidade de nós e tasks por nó
   #SBATCH --nodes=1
   #SBATCH --ntasks-per-node=4
   # Tempo máximo de execução
   #SBATCH --time=01:00:00
   # salvar a saída do job
   #SBATCH --output=mnist_hpc_%j.out

   # Carregar o módulo do Python
   module load python/3.8  

   # Executar o código Python
   srun python atividade_0.py
   ```

   - **Explicação dos parâmetros:**
     - `--job-name`: Nome do job 
     - `--nodes` e `--ntasks-per-node`: Define o número de nós e tarefas por nó (threads/processos).
     - `--time`: Tempo máximo de execução alocado para o job.
     - `--output`: Arquivo onde será salva a saída do job.
     - `module load python/3.8`: Carrega o módulo Python no ambiente do cluster.
     - `srun python atividade_0.py`: Executa o script Python que você desenvolveu na Atividade 0.

3. **Transferir o Código para o Cluster:**

   - Utilize o comando `scp` ou o VS Code configurado para acesso remoto para transferir o script `mnist_slurm_job.sh` e o arquivo `atividade_0.py` para o cluster.

   ```bash
   scp mnist_slurm_job.sh atividade_0.py seu_usuario@cluster_ip:/caminho/desejado/no/cluster/Franky
   ```

#### **2. Executar o Código no Cluster Franky Usando SLURM**

1. **Acessar o Cluster via SSH:**

   - Conecte-se ao cluster utilizando o terminal ou VS Code configurado:
     ```bash
     ssh seu_usuario@cluster_ip
     ```

2. **Submeter o Job SLURM:**

   - Vá até a pasta scratch dentro do seu usuário, o script SLURM e o código Python precisam estar lá para que sejam executados adequadamente.

   - Submeta o job ao SLURM utilizando o comando `sbatch`:
     ```bash
     sbatch mnist_slurm_job.sh
     ```

3. **Monitorar o Status do Job:**

   - Use o comando `squeue` para verificar o status do seu job:
     ```bash
     squeue 
     ```
   - Para visualizar a saída do job, consulte o arquivo `.out` gerado:
     ```bash
     cat mnist_hpc_<job_id>.out
     ```

### Atividade 1: Testando diferentes ambientes (Sua máquina e o Cluster Franky)

   - Compare o tempo de execução do código no cluster (registrado no arquivo `.out`) com o tempo de execução do mesmo código no ambiente local (obtido no relatório da Atividade 0).
   - Registre os resultados obtidos no cluster em um novo relatório.
   - Inclua comparações de tempo, uso de recursos e qualquer outra observação relevante sobre a eficiência computacional entre o ambiente local e o cluster.
   - Discuta como o uso do cluster e SLURM impactou o desempenho do código.
   - A atividade deverá ser entregue pelo Blackboard até as 23h59 do dia da aula.