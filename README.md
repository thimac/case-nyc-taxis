# case-nyc-taxis
Projeto de ingestão e ETL com dados públicos sobre táxis de Nova York (NYC Taxis).

## Sobre o projeto

Este projeto realiza a ingestão de dados brutos, seguida por um processo de ETL (Extração, Transformação e Carga) para limpar, transformar e preparar os dados para análise. O desenvolvimento foi feito utilizando o ambiente Databricks.

## 🛠️ Pré-requisitos

Antes de começar, garanta que você tenha os seguintes pré-requisitos:
*   Python 3.8+
*   Acesso a um ambiente Databricks ou outro que rode o Spark.
*   Um bucket na AWS S3 e as devidas credenciais de acesso (Access Key ID, Secret Access Key e Nome do Bucket).

## ⚙️ Configuração do Ambiente

1.  **Clone o repositório:**
    ```bash
    git clone https://github.com/thimac/case-nyc-taxis.git
    cd case-nyc-taxis
    ```

2.  **Instale as dependências:**
    Para instalar as bibliotecas necessárias, execute o comando abaixo:
    ```bash
    pip install -r requirements.txt
    ```

3.  **Configure as variáveis de ambiente:**
    Este projeto precisa de credenciais para se conectar ao seu bucket no S3. Crie um arquivo chamado `.env` na raiz do projeto.

    Copie o conteúdo do exemplo abaixo para o seu arquivo `.env` e substitua os valores pelas suas credenciais:
    ```
    AWS_ACCESS_KEY_ID="SUA_AWS_ACCESS_KEY_ID"
    AWS_SECRET_ACCESS_KEY="SUA_AWS_SECRET_ACCESS_KEY"
    S3_BUCKET="SEU_S3_BUCKET"
    ```
    > **Atenção:** Arquivo `.env` já está no inserido no `.gitignore`.

## 🚀 Como Executar

A execução do projeto pode variar dependendo da sua versão do Databricks ou outro ambiente.

### Ambiente Databricks (Versão Paga)
Se você estiver utilizando uma versão paga do Databricks, que possui suporte à funcionalidade `dbutils.notebook.run`, a execução pode ser centralizada.
*   Execute o notebook `main.ipynb`. Ele foi projetado para orquestrar e executar sequencialmente os notebooks de ingestão e ETL.

### Ambiente Databricks (Versão Community) ou Outro Ambiente
Caso esteja utilizando a versão Community do Databricks (que não suporta `dbutils.notebook.run`) ou executando os scripts em outro ambiente, será necessário executar cada notebook separadamente, na seguinte ordem:
1.  **Notebook de Ingestão:** Execute primeiro o script responsável pela ingestão dos dados.
2.  **Notebook de ETL:** Após a conclusão da ingestão, execute o script que realiza o processo de ETL.

## 📊 Análise dos Dados

Com os dados disponibilizados para consuma em uma tabela externa, eles estão prontos para serem explorados.

### 1. Análise via Notebook
*   O notebook `analise_consumo.ipynb` contém as respostas para as questões do teste, bem como outras análises exploratórias. Nele, você encontrará os principais insights gerados a partir dos dados processados.

    **Observações sobre a Qualidade dos Dados:**

    Durante a análise, foram identificados alguns pontos que merecem atenção e um aprofundamento futuro para garantir a total acuracidade dos resultados:

    *   **Tratamento de Nulos:** A única transformação de limpeza aplicada durante a transformação  foi a substituição de valores nulos da coluna `passenger_count` pela **mediana** da respectiva coluna.
    *   **Distribuição de Datas:** Foi observado que a distribuição de meses nos dados parece ser maior do que o range de dados que foi ingerido, o que pode indicar um problema na fonte.
    *   **Valores Inconsistentes:**
        *   A coluna `total_amount` apresenta valores negativos, o que é atípico para uma cobrança de corrida e precisa ser investigado.
        *   A coluna `passenger_count` contém valores muito acima da capacidade de um táxi convencional (ex: mais de 6 passageiros), sugerindo erros de digitação ou de sistema.
    *   **Engenharia de Features:** Para análises mais detalhadas, seria benéfico criar novas colunas a partir das existentes. Por exemplo, extrair a data e a hora separadamente da coluna `pickup_datetime` pode facilitar análises de sazonalidade diária ou semanal. A única coluna criada neste projeto foi `mes_pickup`, com o propósito específico de particionar a tabela final para otimizar as consultas.


### 2. Consulta via Tabela Externa (SQL)
O processo de ETL também cria uma tabela externa no metastore do Databricks. Essa tabela aponta diretamente para os dados processados e armazenados no formato Delta Lake em seu bucket S3.

Isso permite que você e sua equipe consultem os dados de forma eficiente usando Spark SQL.

> **Importante: Configurando o Acesso ao S3**
> Para que o Databricks consiga ler os dados da tabela externa, a sessão do Spark precisa ser configurada com as mesmas credenciais do S3 que foram usadas para escrever os dados. Se você estiver em um novo notebook ou em uma nova sessão, precisará configurar o acesso ao S3 antes de executar as consultas SQL.
>
> ```python
> # Exemplo de como configurar a sessão do Spark em um notebook.
> AWS_ACCESS_KEY_ID = "SUA_AWS_ACCESS_KEY_ID"
> AWS_SECRET_ACCESS_KEY = "SUA_AWS_SECRET_ACCESS_KEY"
> 
> spark.conf.set("spark.hadoop.fs.s3a.access.key", AWS_ACCESS_KEY_ID)
> spark.conf.set("spark.hadoop.fs.s3a.secret.key", AWS_SECRET_ACCESS_KEY)
> spark.conf.set("spark.hadoop.fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")
> ```

Uma vez configurado, basta acessar normalmente a tabela `taxi_consumo` via Spark SQL.
