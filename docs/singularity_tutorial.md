# Guia Prático: Como Instalar, Configurar e Usar Singularity (Apptainer)

O Singularity (atualmente mantido como **Apptainer**) é uma plataforma de contêineres criada para ambientes de alta performance (HPC). Ele é o padrão em clusters de pesquisa porque permite rodar contêineres sem precisar de privilégios de administrador (root).

## 1. Instalação (Ubuntu/Debian)

Para instalar as dependências e o software nas distribuições baseadas em Debian/Ubuntu, abra o terminal e execute:

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:apptainer/ppa
sudo apt update
sudo apt install -y apptainer
```

Confirme se a instalação funcionou:
```bash
apptainer --version
```

## 2. Configuração de Cache

Antes de baixar imagens, é vital definir onde o cache será salvo, para evitar que sua partição `/home` fique lotada com imagens pesadas.

```bash
# Crie o diretório de cache em um disco de maior capacidade
mkdir -p /caminho/para/disco_maior/apptainer_cache

# Exporte a variável no seu .bashrc ou .zshrc
echo 'export APPTAINER_CACHEDIR=/caminho/para/disco_maior/apptainer_cache' >> ~/.bashrc
source ~/.bashrc
```

## 3. Imagens Estáticas (.sif) vs. Sandboxes

No Apptainer, existem basicamente duas formas de lidar com imagens:
1. **Imagens Estáticas (`.sif`):** São arquivos únicos e **apenas leitura**. Excelentes para reprodutibilidade final.
2. **Sandboxes:** São contêineres em formato de diretório. Eles permitem a **leitura e escrita**, sendo perfeitos para a fase de testes, onde você precisa instalar pacotes iterativamente.

### Baixando uma Imagem Estática (.sif)
```bash
apptainer pull ubuntu.sif docker://ubuntu:22.04
```

## 4. Trabalhando com Sandboxes (Contêineres Editáveis)

Se você precisa criar um ambiente customizado com ferramentas específicas (como bibliotecas para limpeza de matrizes de expressão, automação ou modelos preditivos), o Sandbox é o caminho.

### Criando um Sandbox
Para criar um sandbox a partir do Docker Hub:
```bash
apptainer build --sandbox meu_ambiente_python docker://ubuntu:22.04
```
*Isso criará uma pasta chamada `meu_ambiente_python` no seu diretório atual, contendo toda a estrutura de pastas de um Linux.*

### Acessando e Modificando o Sandbox (`--writable`)
Para entrar no sandbox com permissão de alterar o sistema (instalar programas), você deve usar a flag `--writable`. Se precisar de privilégios de administrador dentro do contêiner para usar gerenciadores de pacote, adicione o `sudo` ou `--fakeroot` (dependendo da configuração do cluster).

```bash
sudo apptainer shell --writable meu_ambiente_python
```
Uma vez dentro, você terá um terminal interativo. Pode instalar o que precisar:
```bash
# Exemplo de comandos rodando DENTRO do contêiner:
apt update
apt install -y python3 python3-pip
pip install pandas scikit-learn
exit # Para sair do contêiner e voltar para sua máquina
```

## 5. Executando Comandos Diretamente (`exec`)

Depois que seu sandbox ou imagem `.sif` estiver pronto, você não precisa "entrar" nele toda vez que quiser rodar algo. O comando `exec` repassa um comando direto para o contêiner.

Por exemplo, para rodar um script de treinamento de modelo sem abrir o contêiner interativamente:
```bash
apptainer exec meu_ambiente_python python3 treinar_random_forest.py
```

## 6. O Pulo do Gato: Montando Diretórios (`--bind`)

Por padrão, o sistema de arquivos do contêiner é isolado da sua máquina host. Para que o seu script (rodando dentro do contêiner) consiga ler seus arquivos locais e salvar os resultados, você precisa "montar" (linkar) esses diretórios usando a flag `--bind` (ou `-B`).

```bash
# Sintaxe: --bind /pasta/na/sua/maquina:/pasta/no/conteiner
apptainer exec --bind /home/dados_projeto:/dados meu_ambiente_python python3 /dados/script_analise.py --input /dados/matriz_expressao.csv
```
Isso garante que tudo que for lido ou salvo na pasta `/dados` dentro do contêiner será fisicamente armazenado na sua pasta local `/home/dados_projeto`, evitando perda de dados!