# Inicialização e Configuração do dbt

Após configurar o banco de dados e o ambiente, o próximo passo é criar e conectar o seu projeto dbt. No terminal, execute o comando abaixo para iniciar o assistente de configuração:

```bash
dbt init
```

Durante a inicialização, você será solicitado a informar os dados do projeto e a configuração do banco. Substitua os dados entre `< >` pelos detalhes da sua conta do Snowflake:

```text
Enter a name for your project (letters, digits, underscore): eth
Which database would you like to use?
[1] snowflake
Enter a number: 1

account (https://<this_value>.snowflakecomputing.com): <sua_conta_snowflake_ex: abc12345-us-east-1>
user (dev username): <seu_usuario>
[1] password
[2] keypair
[3] sso
[4] workload_identity
Desired authentication type option (enter a number): 1
password (dev password): <sua_senha>
role (dev role): ACCOUNTADMIN
warehouse (warehouse name): compute_wh
database (default database that dbt will build objects in): DBT
schema (default schema that dbt will build objects in): <seu_schema>
threads (1 or more) [1]: 1
```

Isso gerará os arquivos do seu projeto e salvará as credenciais no arquivo `profiles.yml` localmente no seu computador. Em seguida, o dbt irá rodar automaticamente o comando `dbt debug` para validar o projeto e atestar que a conexão com o Snowflake foi bem sucedida!

### Resolução de Problemas (Troubleshooting)
Se você enfrentar algum tipo de problema de conexão após a configuração (como, por exemplo, digitar a senha errada), utilize sempre o comando abaixo para analisar e diagnosticar o erro:

```bash
dbt debug
```

Se precisar corrigir a sua senha ou alguma informação digitada incorretamente no `dbt init`, basta editar o arquivo de configurações gerado. Para quem está usando o **WSL**, esse arquivo (`profiles.yml`) fica salvo no seu diretório de usuário do Linux, geralmente no caminho:
`~/.dbt/profiles.yml` (exemplo: `/home/<seu_usuario>/.dbt/profiles.yml`).
