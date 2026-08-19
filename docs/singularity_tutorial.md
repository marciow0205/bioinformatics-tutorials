# Guia Prático: Como Instalar, Configurar e Usar Singularity (Apptainer)

O Singularity (atualmente conhecido e mantido como **Apptainer**) é uma plataforma de contêineres criada especificamente para ambientes de alta performance (HPC). Diferente do Docker, ele permite que você rode contêineres sem precisar de privilégios de administrador (root), o que o torna o padrão em clusters de pesquisa.

Neste tutorial, vamos cobrir desde a instalação até a execução do seu primeiro contêiner.

## 1. Instalação (Ubuntu/Debian)

A versão de código aberto mantida pela Linux Foundation agora se chama Apptainer. Para instalar as dependências e o software nas distribuições baseadas em Debian/Ubuntu, abra o terminal e execute:

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:apptainer/ppa
sudo apt update
sudo apt install -y apptainer
```

Para confirmar se a instalação foi bem-sucedida, verifique a versão:
```bash
apptainer --version
```

## 2. Configuração Básica

Uma das configurações mais importantes antes de começar a baixar imagens é definir onde o cache será salvo. Contêineres podem ser pesados e rapidamente lotar a sua partição `/home`.

Recomenda-se exportar a variável de cache para um diretório com bastante espaço (como um disco de dados secundário):

```bash
# Crie o diretório de cache
mkdir -p /caminho/para/disco_maior/apptainer_cache

# Adicione a variável ao seu .bashrc ou .zshrc
echo 'export APPTAINER_CACHEDIR=/caminho/para/disco_maior/apptainer_cache' >> ~/.bashrc
source ~/.bashrc
```

## 3. Utilizando Contêineres

### Baixando uma Imagem (`pull`)
Você pode baixar imagens tanto do Docker Hub quanto de repositórios específicos do Apptainer. Vamos baixar uma imagem limpa do Ubuntu:

```bash
apptainer pull ubuntu.sif docker://ubuntu:latest
```
Isso criará um arquivo chamado `ubuntu.sif` (Singularity Image Format) no seu diretório atual. Este arquivo é o seu contêiner encapsulado.

### Acessando o Contêiner de Forma Interativa (`shell`)
Para entrar no contêiner e navegar por ele como se fosse um terminal comum:

```bash
apptainer shell ubuntu.sif
```
*Dica:* Note que o seu usuário dentro do contêiner é exatamente o mesmo usuário da sua máquina host (nada de root automático aqui!).

### Executando Comandos Diretamente (`exec`)
Se você quer apenas rodar um script rápido sem entrar no contêiner de forma interativa, use o `exec`:

```bash
apptainer exec ubuntu.sif python3 script_analise.py
```

## 4. O Pulo do Gato: Montando Diretórios (`--bind`)

Por padrão, o Apptainer isola o sistema de arquivos do contêiner da sua máquina. Se os seus scripts, matrizes de dados e resultados estiverem em pastas específicas, você precisa "montar" essas pastas para dentro do contêiner usando a flag `--bind` (ou `-B`).

```bash
# apptainer exec --bind /pasta/na/maquina:/pasta/no/conteiner imagem.sif comando
apptainer exec --bind /data/analises_2026:/dados ubuntu.sif ls -l /dados
```
Com o comando acima, tudo o que o contêiner ler ou escrever na pasta `/dados` será fisicamente salvo na sua pasta local `/data/analises_2026`. Isso é essencial para processar grandes volumes de dados de forma reprodutível!

