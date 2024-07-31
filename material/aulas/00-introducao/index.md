### Aula 0: Fundamentos de HPC no Processamento de grandes dados

A crescente disponibilidade de grandes volumes de dados e o avanço das aplicações de Inteligência Artificial colocam desafios significativos em termos de processamento e análise. Em cenários onde o tempo e a eficiência são críticos, HPC se torna uma ferramenta essencial para lidar com essas demandas. Ao explorar o potencial do paralelismo e da computação distribuída, o HPC permite que tarefas intensivas, como o treinamento de modelos de IA, sejam realizadas de maneira rápida e eficaz.

Utilizando o dataset MNIST, você executará o pré-processamento e o treinamento de um modelo de IA em um ambiente computacional tradicional (sua máquina). Esta experiência inicial proporcionará uma compreensão das limitações desse ambiente, num futuro não tão distante (próxima aula) faremos otimizações e utilizaremos técnicas de escalabilidade em ambientes de alta performance (O cluster Franky).


1. **Preparação do Ambiente:**
   - **Python 3.x** instalado.
   - Bibliotecas necessárias: `pandas`, `scikit-learn`, `matplotlib`.


2. **Exploração Inicial dos Dados:**
    ```python
      import numpy as np
      from sklearn.datasets import fetch_openml
      import pandas as pd
      import matplotlib.pyplot as plt
      from sklearn.model_selection import train_test_split
      from sklearn.preprocessing import StandardScaler
      from sklearn.linear_model import LogisticRegression
      from sklearn.metrics import accuracy_score
      import time

      # Carregando o dataset MNIST com parser='auto'
      mnist = fetch_openml('mnist_784', parser='auto')

      # Convertendo para DataFrame e convertendo os dados para tipo float
      df = pd.DataFrame(mnist.data.astype(np.float32), columns=mnist.feature_names)
      df['target'] = mnist.target.astype(int)  # Converter o target para int, se necessário

      # Verificando o tipo de dados
      print(df.dtypes)

      # Visualizando algumas amostras
      fig, axes = plt.subplots(1, 5, figsize=(10, 3))
      for i, ax in enumerate(axes):
         ax.imshow(df.iloc[i, :-1].values.reshape(28, 28), cmap='gray')
         ax.set_title(f"Label: {df['target'][i]}")
         ax.axis('off')
      plt.show()

    ```

Vamos realizar um pré-processamento simples, incluindo normalização dos dados e divisão em conjuntos de treino e teste.

    ```python
   # Separando features e target
   X = df.drop('target', axis=1)
   y = df['target'].astype('int')

   # Dividindo o dataset em treino e teste
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

   # Normalizando os dados
   scaler = StandardScaler()
   X_train = scaler.fit_transform(X_train)
   X_test = scaler.transform(X_test)

   print(X_train.size, X_test.size)

    ```

Vamos agora treinar um modelo de classificação utilizando regressão logística e medir o tempo de execução.


    ```python
      # Inicializando o modelo de Regressão Logística
      model = LogisticRegression(max_iter=1000, solver='lbfgs')

      # Medindo o tempo de execução
      start_time = time.time()
      model.fit(X_train, y_train)
      end_time = time.time()

      # Avaliando o modelo
      y_pred = model.predict(X_test)
      accuracy = accuracy_score(y_test, y_pred)

      print(f"Acurácia do modelo: {accuracy:.2f}")
      print(f"Tempo de execução para treinamento: {end_time - start_time:.2f} segundos")
    ```


### Solucionadores Disponíveis no `LogisticRegression`

O `scikit-learn` oferece vários tipos de solucionadores para a classe `LogisticRegression`. Cada um desses solucionadores pode ter um desempenho diferente dependendo da natureza dos dados, da escala do problema e dos recursos computacionais disponíveis.

Essa é a lista dos solucionadores disponíveis:

1. **`liblinear`**:
   - **Descrição**: Um solucionador baseado em abordagem de coordenação, bom para pequenos datasets ou quando há um pequeno número de features.
   - **Vantagens**: Funciona bem para problemas binários e de múltiplas classes com regularização L1 ou L2.
   - **Limitações**: Não escala bem com o número de dados ou features.

2. **`newton-cg`**:
   - **Descrição**: Um solucionador que utiliza o método de Newton conjugado para otimização.
   - **Vantagens**: Eficiente para problemas de regressão logística com grande número de features.
   - **Limitações**: Pode ser mais lento que outros solucionadores para grandes datasets.

3. **`lbfgs`**:
   - **Descrição**: Um solucionador baseado no algoritmo de otimização BFGS limitado.
   - **Vantagens**: Funciona bem para problemas de múltiplas classes e é eficiente em grandes datasets.
   - **Limitações**: Pode não convergir para dados mal escalonados.

4. **`sag`**:
   - **Descrição**: Um solucionador que utiliza o método de gradiente estocástico agregado.
   - **Vantagens**: Escala bem com grandes datasets e funciona bem para problemas com regularização L2.
   - **Limitações**: Não suporta regularização L1.

5. **`saga`**:
   - **Descrição**: Uma versão aprimorada do `sag` que suporta regularização L1 e L2, além de ser eficaz para grandes datasets.
   - **Vantagens**: Funciona bem para problemas de múltiplas classes, é robusto para diferentes tipos de regularização e escalável.
   - **Limitações**: Pode ser mais lento que `lbfgs` para pequenos datasets.

### Atividade 0: Testando Diferentes Solucionadores

1. **Testar Diferentes Solucionadores:**
   - treinar o modelo de regressão logística utilizando cada um dos solucionadores disponíveis.
   - Para cada solucionador, medir o tempo de execução, verificar se o modelo convergiu e registrar a acurácia do modelo.

2. **Documentar os Resultados:**
   - Crie um relatório que inclua:
     - Acurácia do modelo para cada solucionador.
     - Tempo de execução para o treinamento do modelo com cada solucionador.

3. **Análise Comparativa:**
   - Faça uma análise comparativa para determinar qual solucionador foi mais eficiente em termos de tempo e qual proporcionou a melhor acurácia. 

4. **Entrega da atividade**
   - A atividade deverá ser entregue pelo Blackboard até as 23h59 do dia da aula.