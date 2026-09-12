# Configuração do Ambiente Local (Linux / WSL)

Para a etapa de modelagem, utilizaremos o **dbt Core** via linha de comando. É altamente recomendável realizar a instalação e execução das ferramentas de dados em um ambiente **Linux**.

**Por que usar Linux em Projetos de Engenharia de Dados?**
* **Compatibilidade:** A grande maioria das ferramentas de dados (incluindo o dbt e o ecossistema Python) é desenvolvida com foco primário em sistemas Unix (Linux/macOS). Isso evita problemas comuns com caminhos de arquivos e dependências no Windows.
* **Mercado de Trabalho:** Os servidores de produção na nuvem utilizam Linux quase em sua totalidade. Praticar nesse ambiente te prepara melhor para cenários reais.
* **Usuários Windows:** A melhor forma de utilizar Linux sem sair do Windows é instalar o **WSL (Windows Subsystem for Linux)**, preferencialmente utilizando a distribuição Ubuntu. Assim, você obtém um terminal Linux nativo integrado ao seu sistema operacional.

## 1. Instalação do Python e UV (Gerenciador de Pacotes)
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

## 2. Criação do Repositório e Controle de Versão (Git)
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

## 3. Ambiente Virtual e Instalação do dbt-snowflake
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
