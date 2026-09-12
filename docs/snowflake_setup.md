# Configuração do Snowflake

Esta seção documenta o processo de extração dos dados públicos da blockchain Ethereum armazenados na AWS S3 e o carregamento para o Snowflake.

## 1. Criação da Conta no Snowflake
Para este laboratório, o primeiro passo foi criar uma conta *Trial* (gratuita de 30 dias) no [Snowflake](https://signup.snowflake.com/).
Após criar e acessar a sua conta, abra uma nova planilha (Worksheet) de SQL para executar os comandos abaixo.

## 2. Criação do Banco de Dados e Schema
No Snowflake, os dados ficam organizados dentro de Bancos de Dados e, internamente, em Schemas. Criamos o ambiente para os dados de Ethereum (`ETH`).

```sql
-- Cria o banco de dados chamado ETH
CREATE DATABASE ETH;
		
-- Cria um schema específico chamado ETH_SCHEMA dentro do banco ETH
CREATE SCHEMA ETH.ETH_SCHEMA;
```

## 3. Configuração dos "Stages" (Conexão com a AWS S3)
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

## 4. Criação das Tabelas Raw no Snowflake
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

## 5. Carregamento dos Dados (COPY INTO)
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
