# Jornada de Aprendizado DBT + Snowflake: Analisando Transações Ethereum

Bem-vindo a este repositório! A missão principal aqui é **documentar uma jornada de aprendizado do dbt (data build tool)** voltada para a preparação para certificações em Analytics Engineering.

Neste projeto inicial, o escopo é obter dados públicos da blockchain Ethereum armazenados em um bucket público do Amazon S3 (AWS), carregá-los no Snowflake (Data Warehouse) e, posteriormente, modelar e transformar esses dados utilizando o dbt.

Este guia foi desenhado para ser um passo a passo detalhado, permitindo que até mesmo leigos consigam replicar o ambiente e os testes práticos.

---

## 🏗️ Arquitetura do Projeto

1. **Fonte de Dados (Raw):** AWS S3 (Bucket público com dados da blockchain Ethereum em formato Parquet).
2. **Data Warehouse (Armazenamento e Computação):** Snowflake.
3. **Transformação de Dados:** dbt (Data Build Tool) - *Em andamento*.

---

## 📚 Documentação e Passo a Passo

Para facilitar a leitura e o acompanhamento, o processo completo foi dividido em seções menores e detalhadas. Siga a ordem abaixo para replicar o projeto:

### 1. Preparação da Camada de Dados (Snowflake)
Aqui demonstramos como preparar o seu banco de dados analítico e ingerir os dados do Ethereum diretamente da AWS S3 para o Snowflake.
👉 **[Ir para Configuração do Snowflake](docs/snowflake_setup.md)**

### 2. Preparação do Ambiente de Transformação (dbt + Python)
Neste documento, exploramos como preparar um ambiente de desenvolvimento robusto utilizando ferramentas de mercado (Linux/WSL, Python, gerenciador UV) e a instalação do dbt.
👉 **[Ir para Configuração do Ambiente Local](docs/local_env_setup.md)**

---

## 🚀 Próximos Passos
Com o Snowflake populado e o ambiente local Python (uv) pronto com o dbt instalado, o próximo passo consiste em inicializar o projeto dbt (`dbt init`), configurar os perfis de conexão com a nossa conta do Snowflake (arquivo `profiles.yml`) e iniciar a criação dos modelos analíticos!
