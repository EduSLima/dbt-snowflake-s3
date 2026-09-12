# Jornada de Aprendizado DBT + Snowflake: Analisando Transações Ethereum

Bem-vindo a este repositório! A missão principal aqui é **documentar uma jornada de aprendizado do dbt (data build tool)** voltada para a preparação para certificações.

Neste projeto inicial, o escopo é obter dados públicos da blockchain Ethereum armazenados em um bucket público do Amazon S3 (AWS), carregá-los no Snowflake (Data Warehouse) e, posteriormente, modelar e transformar esses dados utilizando o dbt.

Este guia foi desenhado para ser um passo a passo detalhado, permitindo que até mesmo leigos consigam replicar o ambiente e os testes.

---

## Arquitetura Inicial

1. **Fonte de Dados (Raw):** AWS S3 (Bucket público com dados da blockchain Ethereum em formato Parquet).
2. **Data Warehouse (Armazenamento e Computação):** Snowflake.
3. **Transformação de Dados:** dbt (Data Build Tool) - *Em Breve*.

---

## Passo a Passo para Replicação (Até o Momento)

### 1. Criação da Conta no Snowflake
Para este laboratório, o primeiro passo foi criar uma conta *Trial* (gratuita de 30 dias) no [Snowflake](https://signup.snowflake.com/).
Após criar e acessar a sua conta, abra uma nova planilha (Worksheet) de SQL para executar os comandos abaixo.

### 2. Criação do Banco de Dados e Schema
No Snowflake, os dados ficam organizados dentro de Bancos de Dados e, internamente, em Schemas. Criamos o ambiente para os dados de Ethereum (`ETH`).

```sql
-- Cria o banco de dados chamado ETH
CREATE DATABASE ETH;
		
-- Cria um schema específico chamado ETH_SCHEMA dentro do banco ETH
CREATE SCHEMA ETH.ETH_SCHEMA;
```

### 3. Configuração dos "Stages" (Conexão com a AWS S3)
Um *Stage* (estágio) no Snowflake é uma referência para o local onde os arquivos de dados estão armazenados (neste caso, na nuvem da AWS). Apontamos o Snowflake diretamente para o bucket público de dados da rede Ethereum.

```sql
-- Stage genérico de teste para visualizar os arquivos
CREATE STAGE ETH.ETH_SCHEMA.TEST
    url = 's3://aws-public-blockchain/v1.0/eth/'
    FILE_FORMAT = (TYPE = 'PARQUET');

-- Comando para listar os arquivos existentes neste bucket de teste
LIST @ETH.ETH_SCHEMA.TEST;

-- Criando stages específicos para os domínios de dados (Contratos, Transferências e Transações)
CREATE OR REPLACE STAGE ETH.ETH_SCHEMA.CONTRACTS_STAGE
    url='s3://aws-public-blockchain/v1.0/eth/contracts'
    FILE_FORMAT= (TYPE= 'PARQUET');

CREATE OR REPLACE STAGE ETH.ETH_SCHEMA.TOKEN_TRANSFERS
    url='s3://aws-public-blockchain/v1.0/eth/token_transfers'
    FILE_FORMAT= (TYPE= 'PARQUET');

CREATE OR REPLACE STAGE ETH.ETH_SCHEMA.TRANSACTIONS
    url='s3://aws-public-blockchain/v1.0/eth/transactions'
    FILE_FORMAT= (TYPE= 'PARQUET');
```

### 4. Criação das Tabelas Raw no Snowflake
Agora, definimos a estrutura (as colunas e tipos de dados) onde os arquivos em Parquet da nuvem serão armazenados dentro do nosso Data Warehouse.

```sql
-- Tabela para Smart Contracts
CREATE OR REPLACE TABLE ETH.ETH_SCHEMA.CONTRACTS (
    address string,
    block_hash string,
    block_number int,
    block_timestamp TIMESTAMP,
    bytecode string,
    date date,
    last_modified TIMESTAMP
);

-- Tabela para Transferências de Tokens
CREATE OR REPLACE TABLE ETH.ETH_SCHEMA.TOKEN_TRANSFERS (
    BLOCK_HASH STRING,
    BLOCK_NUMBER  INT,
    BLOCK_TIMESTAMP TIMESTAMP,
    DATE DATE,
    FROM_ADDRESS STRING,
    TO_ADDRESS STRING,
    LAST_MODIFIED TIMESTAMP,
    LOG_INDEX INT,
    TOKEN_ADDRESS STRING,
    TRANSACTION_HASH STRING,
    VALUE FLOAT
);

-- Tabela para as Transações
CREATE OR REPLACE TABLE ETH.ETH_SCHEMA.TRANSACTIONS (
    BLOCK_HASH STRING,
    BLOCK_NUMBER INT,
    BLOCK_TIMESTAMP TIMESTAMP,
    DATE DATE,
    FROM_ADDRESS  STRING,
    GAS INT,
    GAS_PRICE STRING,
    HASH STRING,
    INPUT STRING,
    LAST_MODIFIED TIMESTAMP,
    MAX_FEE_PER_GAS STRING,
    MAX_PRIORITY_FEE_PER_GAS  STRING,
    NONCE STRING,
    RECEIPT_CONTRACT_ADDRESS STRING,
    RECEIPT_CUMULATIVE_GAS_USED INT,
    RECEIPT_EFFECTIVE_GAS_PRICE STRING,
    RECEIPT_GAS_USED INT,
    RECEIPT_STATUS INT,
    TO_ADDRESS STRING,
    TRANSACTION_INDEX INT,
    TRANSACTION_TYPE INT,
    VALUE FLOAT
);
```

### 5. Carregamento dos Dados (COPY INTO)
Com os buckets apontados e as tabelas criadas, usamos o comando `COPY INTO` para trazer fisicamente os dados do bucket para dentro das nossas tabelas no Snowflake. 
Para não trazer toda a base de dados (que é gigantesca), configuramos um filtro dinâmico (`PATTERN`) que busca apenas os arquivos referentes ao mês passado em relação à data atual (últimos 30 dias).

```sql
-- Configuração da variável que define a máscara do mês desejado (ex: .*date=2024-04.*)
SET CURRENT_MONTH_PATTERN = (Select CONCAT('.*date=',TO_VARCHAR(CURRENT_DATE()-30,'YYYY-MM'),'.*'));

-- Visualizar o valor da variável recém-criada
SELECT $CURRENT_MONTH_PATTERN;

-- Copiando dados de Contratos
COPY INTO ETH.ETH_SCHEMA.CONTRACTS
FROM (
    select
        t.$1:address,
        t.$1:block_hash,
        t.$1:block_number,
        t.$1:block_timestamp,
        t.$1:bytecode,
        t.$1:date,
        t.$1:last_modified
    from @ETH.ETH_SCHEMA.CONTRACTS_STAGE t
)
PATTERN = $CURRENT_MONTH_PATTERN;

-- Copiando dados de Transferência de Tokens
COPY INTO ETH.ETH_SCHEMA.TOKEN_TRANSFERS
FROM (
    select
        t.$1:block_hash,
        t.$1:block_number,
        t.$1:block_timestamp,
        t.$1:date,
        t.$1:from_address,
        t.$1:to_address,
        t.$1:last_modified,
        t.$1:log_index,
        t.$1:token_address,
        t.$1:transaction_hash,
        t.$1:value
    from @ETH.ETH_SCHEMA.TOKEN_TRANSFERS t
)
PATTERN = $CURRENT_MONTH_PATTERN;

-- Copiando dados de Transações
COPY INTO ETH.ETH_SCHEMA.TRANSACTIONS
FROM (
    select
        t.$1:block_hash,
        t.$1:block_number,
        t.$1:block_timestamp,
        t.$1:date,
        t.$1:from_address,
        t.$1:gas,
        t.$1:gas_price,
        t.$1:hash,
        t.$1:input,
        t.$1:last_modified,
        t.$1:max_fee_per_gas,
        t.$1:max_priority_fee_per_gas,
        t.$1:nonce,
        t.$1:receipt_contract_address,
        t.$1:receipt_cumulative_gas_used,
        t.$1:receipt_effective_gas_price,
        t.$1:receipt_gas_used,
        t.$1:receipt_status,
        t.$1:to_address,
        t.$1:transaction_index,
        t.$1:transaction_type,
        t.$1:value
    from @ETH.ETH_SCHEMA.TRANSACTIONS t
)
PATTERN = $CURRENT_MONTH_PATTERN;         
```

---

## 6. Configuração do Ambiente Local (Linux / WSL)

Para a etapa de modelagem, utilizaremos o **dbt Core** via linha de comando. É altamente recomendável realizar a instalação e execução das ferramentas de dados em um ambiente **Linux**.

**Por que usar Linux em Projetos de Engenharia de Dados?**
* **Compatibilidade:** A grande maioria das ferramentas de dados (incluindo o dbt e o ecossistema Python) é desenvolvida com foco primário em sistemas Unix (Linux/macOS). Isso evita problemas comuns com caminhos de arquivos e dependências no Windows.
* **Mercado de Trabalho:** Os servidores de produção na nuvem utilizam Linux quase em sua totalidade. Praticar nesse ambiente te prepara melhor para cenários reais.
* **Usuários Windows:** A melhor forma de utilizar Linux sem sair do Windows é instalar o **WSL (Windows Subsystem for Linux)**, preferencialmente utilizando a distribuição Ubuntu. Assim, você obtém um terminal Linux nativo integrado ao seu sistema operacional.

### 6.1. Instalação do Python e UV (Gerenciador de Pacotes)
Neste projeto, utilizamos o **uv**, um gerenciador de pacotes e ambientes Python extremamente rápido, escrito em Rust.

No terminal do seu Ubuntu (ou WSL), atualize os pacotes do sistema e garanta que o Python está instalado:
```bash
sudo apt update
sudo apt install python3-full
```

Em seguida, instale o `uv` via script oficial:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
*Dica: Após a instalação, execute `source ~/.zshrc` (ou `.bashrc`, se não usar o ZSH) ou reinicie o terminal para que o comando `uv` fique disponível.*

### 6.2. Criação do Repositório e Controle de Versão (Git)
Se você não fez a clonagem do projeto e está criando do zero, crie uma pasta, navegue até ela e configure o Git.

```bash
mkdir dbt-snowflake-s3
cd dbt-snowflake-s3

# Inicialização do Git
git init
echo "# dbt-snowflake-s3" >> README.md
git add README.md
git commit -m "first commit"

# Configuração de identidade (substitua pelos seus dados)
git config --global user.email "seu.email@exemplo.com"
git config --global user.name "Seu Nome"

# (Opcional) Gerar e exibir chave SSH para conectar com o GitHub com segurança
ssh-keygen -t ed25519 -C "seu.email@exemplo.com"
cat ~/.ssh/id_ed25519.pub

# Vincular ao seu repositório remoto no GitHub
git branch -M main
git remote add origin git@github.com:SeuUsuario/dbt-snowflake-s3.git
git push -u origin main
```

### 6.3. Ambiente Virtual e Instalação do dbt-snowflake
Crie um ambiente virtual para isolar as bibliotecas do projeto da sua máquina. O pacote `dbt-snowflake` instalará automaticamente o `dbt-core` e as dependências necessárias para a comunicação com o Snowflake.

```bash
# Criar ambiente virtual apontando para o Python 3.12 (ou a versão instalada)
uv venv --python 3.12

# Ativar o ambiente virtual
source .venv/bin/activate

# Instalar o dbt-snowflake
uv pip install dbt-snowflake
```
*(Quando o ambiente virtual está ativo, geralmente o prefixo `(.venv)` aparece no início do terminal).*

Verifique se a instalação ocorreu com sucesso checando a versão do dbt:
```bash
dbt --version
```

---

## Próximos Passos
Com o Snowflake populado e o ambiente local Python (uv) pronto com o dbt instalado, o próximo passo consiste em inicializar o projeto dbt (`dbt init`), configurar os perfis de conexão com a nossa conta do Snowflake (arquivo `profiles.yml`) e iniciar a criação dos modelos analíticos!
