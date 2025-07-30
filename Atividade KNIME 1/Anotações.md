## Visão Geral do Fluxo

- Aula 29/07/2025 - Terça Feira
- Erik Nathan | enob@cesar.school

### 1. Leitura e Visualização dos Dados

* **`CSV Reader` (Ler Dataset SalarioData.csv):** Este é o ponto de partida. Ele lê o seu arquivo `SalarioData.csv` e o carrega para dentro do KNIME como uma tabela. A saída deste nó é a tabela de dados brutos com as colunas `YearsExperience` e `Salary`.

* **`Line Plot` e `Scatter Plot`:** Estes nós estão conectados diretamente ao leitor de CSV para a **Análise Exploratória de Dados (EDA)**.
    * **`Scatter Plot` (Gráfico de Dispersão):** É a visualização mais importante aqui. Ele permite que você coloque `YearsExperience` no eixo X e `Salary` no eixo Y. Isso mostra visualmente a relação entre as duas variáveis. Você esperaria ver pontos que formam uma linha ascendente, indicando que quanto maior a experiência, maior o salário.
    * **`Line Plot` (Gráfico de Linha):** Para estes dados, um gráfico de linha pode não ser a melhor visualização, a menos que os dados estejam perfeitamente ordenados por experiência. Geralmente, o gráfico de dispersão é mais adequado para entender a correlação.

### 2. Pré-processamento e Preparação

* **`Normalizer` (Normalização dos Valores Numéricos):** Este nó ajusta a escala dos seus dados numéricos. Muitos algoritmos de machine learning funcionam melhor quando os dados estão em uma escala semelhante (por exemplo, todos entre 0 e 1). Ele transforma as colunas `YearsExperience` e `Salary` para essa nova escala.
    * **Importante:** Este nó tem duas saídas. A primeira (em cima) são os dados normalizados. A segunda (embaixo, o quadrado azul) é o **"modelo de normalização"**, que guarda as informações de como a normalização foi feita (ex: os valores mínimos e máximos originais). Isso é crucial para reverter o processo mais tarde.

* **`Table Partitioner` (Particionador de Tabela):** Este é um passo fundamental em qualquer projeto de machine learning. Ele divide seus dados em dois conjuntos distintos:
    1.  **Conjunto de Treinamento (saída de cima):** Geralmente a maior parte dos dados (ex: 80%). O modelo vai "aprender" a partir destes dados.
    2.  **Conjunto de Teste (saída de baixo):** O restante dos dados (ex: 20%). O modelo nunca vê esses dados durante o treinamento. Eles são usados no final para avaliar o desempenho do modelo em dados "novos".

### 3. Treinamento e Predição do Modelo

* **`Linear Regression Learner` (Treinamento de Regressão Linear):** Este é o cérebro da operação. Ele recebe o **conjunto de treinamento** e aplica o algoritmo de regressão linear para encontrar a melhor linha reta que descreve a relação entre `YearsExperience` (variável independente) e `Salary` (variável alvo). A saída não é uma tabela de dados, mas sim um **modelo treinado** (o quadrado azul), que contém a fórmula matemática (ex: $Salário = (m \times AnosDeExperiência) + b$).

* **`Regression Predictor` (Previsor de Regressão):** Este nó faz duas coisas:
    1.  Recebe o **modelo treinado** do nó anterior.
    2.  Recebe o **conjunto de teste** do `Table Partitioner`.
    Ele então aplica a fórmula do modelo aos dados de teste para prever o salário para cada entrada. A saída é uma tabela que contém os dados de teste originais mais uma nova coluna com os salários previstos.

### 4. Pós-processamento e Avaliação

Esta parte final do fluxo serve para comparar os salários reais com os salários previstos e calcular a performance do modelo. Como as previsões foram feitas em dados normalizados, precisamos primeiro revertê-los para a escala original (em reais).

* **`Reference Column Splitter`, `Column Renamer`, `Denormalizer` (x2) e `Column Appender`:** Este conjunto de nós parece complexo, mas seu objetivo é simples: **desnormalizar** os valores.
    1.  `Reference Column Splitter` e `Column Renamer`: Provavelmente estão separando a coluna de salário real e a coluna de salário previsto para tratá-las de forma independente.
    2.  **`Denormalizer` (x2):** Cada um desses nós recebe uma das colunas (a real e a prevista) e o **modelo de normalização** (o quadrado azul lá do começo). Eles usam esse modelo para reverter os valores para sua escala original (valores monetários de salário).
    3.  `Column Appender`: Junta as colunas desnormalizadas de volta em uma única tabela. Agora você tem uma tabela final com o "Salário Real" e o "Salário Previsto" lado a lado, em valores compreensíveis.

* **`Numeric Scorer` (Avaliador Numérico):** Este é o nó final. Ele recebe a tabela com os valores reais e previstos e calcula métricas de erro estatístico para avaliar a performance do modelo. Ele responde à pergunta: "Quão bom é o meu modelo?". As métricas comuns para regressão que ele calcula são:
    * **$R^2$ (R-quadrado):** Indica a proporção da variação na variável dependente (salário) que é previsível a partir da variável independente (experiência). Quanto mais perto de 1, melhor.
    * **MAE (Mean Absolute Error):** O erro médio absoluto das previsões, na mesma unidade da variável (neste caso, em reais).
    * **RMSE (Root Mean Squared Error):** A raiz do erro quadrático médio. Também está na unidade da variável e penaliza erros maiores com mais intensidade.