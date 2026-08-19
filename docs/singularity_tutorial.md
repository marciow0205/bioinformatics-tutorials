# Guia Prático: Como Instalar, Configurar e Usar Singularity (Apptainer)

> **Documentação oficial:** https://apptainer.org/docs/user/latest/

## O que é o Apptainer?

O **Singularity**, atualmente mantido como **Apptainer**, é uma tecnologia de contêineres amplamente utilizada em clusters, servidores de pesquisa e ambientes de computação científica.

Uma das principais diferenças em relação ao Docker está no modelo de permissões:

- No **Docker**, dependendo da configuração, os processos podem ser executados com privilégios elevados, o que exige atenção especial em servidores compartilhados.
- No **Apptainer**, o usuário normalmente mantém suas permissões dentro do contêiner. Em outras palavras, entrar em um contêiner não transforma automaticamente o usuário em `root`.

Isso é especialmente útil em laboratórios e servidores compartilhados, pois ajuda a manter o isolamento do ambiente sem alterar as permissões do usuário no sistema hospedeiro.

---

# 1. Instalação no Ubuntu/Debian

Em distribuições baseadas em Debian/Ubuntu, uma forma prática de instalar o Apptainer é utilizar o repositório oficial disponibilizado pelo projeto.

### 1.1. Atualize os pacotes

```bash
sudo apt update
```

### 1.2. Instale as dependências necessárias

```bash
sudo apt install -y software-properties-common
```

### 1.3. Adicione o repositório do Apptainer

```bash
sudo add-apt-repository -y ppa:apptainer/ppa
```

Depois, atualize novamente a lista de pacotes:

```bash
sudo apt update
```

### 1.4. Instale o Apptainer

```bash
sudo apt install -y apptainer
```

### 1.5. Verifique a instalação

Execute:

```bash
apptainer --version
```

Uma saída possível é:

```text
apptainer version 1.2.5
```

> **Observação:** a versão exibida pode ser diferente da apresentada neste exemplo.

---

# 2. Configuração do Cache

## Por que configurar o cache?

Quando você baixa imagens ou constrói contêineres, o Apptainer utiliza um diretório de cache, normalmente localizado em:

```text
~/.apptainer/cache
```

Em servidores de pesquisa, isso pode ser um problema porque o diretório `/home` frequentemente possui uma **quota de armazenamento**.

Se você trabalhar com várias imagens grandes — por exemplo, ambientes contendo Python, R, Bioconductor ou ferramentas de RNA-Seq — o cache pode crescer rapidamente.

Uma solução é direcionar o cache para um disco com maior capacidade de armazenamento.

## 2.1. Crie o diretório de cache

Substitua `seu_usuario` pelo seu usuário ou pelo caminho apropriado no servidor:

```bash
mkdir -p /mnt/dados_lab/seu_usuario/apptainer_cache
```

## 2.2. Configure a variável de ambiente

Adicione a configuração ao `.bashrc`:

```bash
echo 'export APPTAINER_CACHEDIR=/mnt/dados_lab/seu_usuario/apptainer_cache' >> ~/.bashrc
```

## 2.3. Recarregue o `.bashrc`

```bash
source ~/.bashrc
```

## 2.4. Verifique a configuração

```bash
echo $APPTAINER_CACHEDIR
```

A saída esperada será semelhante a:

```text
/mnt/dados_lab/seu_usuario/apptainer_cache
```

> **Boa prática:** confirme se o diretório escolhido possui espaço suficiente e se o seu usuário possui permissão de leitura e escrita nele.

---

# 3. Imagens `.sif` vs. Sandboxes

O Apptainer trabalha principalmente com dois formatos importantes: **imagens `.sif`** e **sandboxes**.

## 3.1. Imagem estática `.sif`

Uma imagem `.sif` (*Singularity Image Format*) normalmente é distribuída como um **único arquivo** e é utilizada como um ambiente de execução mais estável e reprodutível.

Características:

- arquivo único;
- adequada para distribuição;
- normalmente utilizada em modo somente leitura;
- excelente para pipelines finalizados;
- facilita a reprodução de análises;
- pode ser compartilhada junto com um trabalho científico.

Por exemplo:

```text
ambiente_final.sif
```

## 3.2. Sandbox

Um **sandbox** é um contêiner armazenado como uma estrutura de diretórios.

Características:

- possui estrutura semelhante a um sistema Linux;
- pode ser modificado;
- é útil durante a fase de desenvolvimento;
- permite testar instalações e alterações de forma iterativa.

### Comparação rápida

| Característica | `.sif` | Sandbox |
|---|---|---|
| Formato | Arquivo único | Diretório |
| Modificação | Não é o fluxo normal | Sim |
| Desenvolvimento | Menos conveniente | Ideal |
| Distribuição | Excelente | Menos prática |
| Reprodutibilidade | Excelente quando finalizado | Mais sujeita a alterações |

> **Regra prática:** desenvolva no **sandbox** e, quando o ambiente estiver funcionando, transforme-o em uma imagem `.sif` para uso final.

---

# 4. Baixando uma Imagem `.sif`

Podemos obter uma imagem diretamente de uma fonte compatível, como o Docker Hub.

Por exemplo, para baixar o Ubuntu 22.04:

```bash
apptainer pull ambiente_base.sif docker://ubuntu:22.04
```

O comando fará o download da imagem e criará:

```text
ambiente_base.sif
```

no diretório atual.

Depois, você poderá verificar o arquivo com:

```bash
ls -lh ambiente_base.sif
```

> **Importante:** uma imagem Ubuntu básica não contém automaticamente Python, R, Bioconductor ou outras ferramentas científicas. Essas ferramentas precisam ser instaladas ou incorporadas ao ambiente.

---

# 5. Criando um Sandbox Editável

Imagine que você queira montar um ambiente para um modelo de Machine Learning, utilizando uma assinatura de 148 genes e bibliotecas como `pandas` e `scikit-learn`.

Nesse caso, um sandbox pode ser útil durante o desenvolvimento.

## 5.1. Criar o sandbox

```bash
apptainer build --sandbox env_machine_learning docker://ubuntu:22.04
```

Em vez de gerar um único arquivo `.sif`, o comando criará um diretório:

```text
env_machine_learning/
```

Dentro dele haverá uma estrutura semelhante à de um sistema Linux:

```text
env_machine_learning/
├── bin/
├── etc/
├── usr/
├── var/
└── ...
```

---

# 6. Acessando e Modificando o Sandbox

Para abrir um sandbox com possibilidade de escrita, utilizamos:

```bash
apptainer shell --writable env_machine_learning
```

Dependendo da configuração do servidor, determinadas operações podem exigir privilégios adicionais.

Em ambientes nos quais o `fakeroot` está habilitado, por exemplo:

```bash
apptainer shell --writable --fakeroot env_machine_learning
```

Se a configuração do servidor exigir `sudo`, pode ser necessário utilizar:

```bash
sudo apptainer shell --writable env_machine_learning
```

Ao entrar no contêiner, você verá algo semelhante a:

```text
Apptainer>
```

Isso indica que você está executando comandos dentro do contêiner.

## 6.1. Instalar ferramentas

Por exemplo:

```bash
Apptainer> apt update
Apptainer> apt install -y python3-pip
Apptainer> pip install pandas scikit-learn
```

Quando terminar:

```bash
Apptainer> exit
```

O comando `exit` encerra o shell do contêiner e retorna ao servidor.

> **Atenção:** os comandos de instalação acima são apenas um exemplo. Em um ambiente científico real, é importante controlar versões das bibliotecas para garantir reprodutibilidade.

---

# 7. Executando Comandos com `exec`

Depois que o ambiente estiver pronto, você não precisa entrar no contêiner manualmente toda vez.

O comando:

```bash
apptainer exec
```

permite executar diretamente um programa dentro do contêiner.

Por exemplo:

```bash
apptainer exec env_machine_learning python3 treinar_modelo.py
```

Nesse caso:

1. o Apptainer inicia o ambiente;
2. executa `python3`;
3. roda `treinar_modelo.py`;
4. exibe a saída no terminal;
5. encerra o processo do contêiner quando o comando termina.

Esse modelo é particularmente útil para pipelines automatizados.

---

# 8. O Conceito Mais Importante: `--bind`

Um dos conceitos fundamentais do Apptainer é o **bind mount**.

Imagine que seus dados estejam no servidor em:

```text
/mnt/dados_rnaseq/projeto_tcga
```

e seus resultados devam ser salvos em:

```text
/mnt/resultados_lab/analises_2026
```

Você pode disponibilizar esses diretórios dentro do contêiner utilizando `--bind` ou `-B`.

A sintaxe básica é:

```text
--bind /caminho/no/servidor:/caminho/dentro/do/contêiner
```

Por exemplo:

```bash
apptainer exec \
  --bind /mnt/dados_rnaseq/projeto_tcga:/input_dados \
  --bind /mnt/resultados_lab/analises_2026:/output_resultados \
  env_machine_learning \
  python3 /input_dados/scripts/treinar_random_forest.py \
    --matriz /input_dados/matriz_expressao_tcga.csv \
    --out /output_resultados/modelo_148_genes.pkl
```

## 8.1. O que está acontecendo?

No servidor, temos:

```text
/mnt/dados_rnaseq/projeto_tcga
```

Dentro do contêiner, esse mesmo diretório aparecerá como:

```text
/input_dados
```

Da mesma forma:

```text
/mnt/resultados_lab/analises_2026
```

será disponibilizado dentro do contêiner como:

```text
/output_resultados
```

Podemos visualizar o processo assim:

```text
SERVIDOR
│
├── /mnt/dados_rnaseq/projeto_tcga
│       │
│       └──────────────► /input_dados
│
└── /mnt/resultados_lab/analises_2026
        │
        └──────────────► /output_resultados

                     CONTÊINER
```

## 8.2. Por que isso é importante?

Isso permite separar claramente:

- **ambiente computacional** → contêiner;
- **dados de entrada** → servidor;
- **resultados** → servidor.

Assim, o contêiner pode ser substituído ou recriado sem perder os dados.

Por exemplo:

```text
Contêiner
├── Python
├── pandas
├── scikit-learn
└── ferramentas do pipeline

Servidor
├── matriz de expressão
├── scripts
├── arquivos FASTQ
├── resultados
└── modelos treinados
```

O arquivo:

```text
modelo_148_genes.pkl
```

será salvo diretamente no servidor em:

```text
/mnt/resultados_lab/analises_2026/modelo_148_genes.pkl
```

---

# 9. Congelando o Ambiente: Sandbox → `.sif`

Durante o desenvolvimento, o sandbox é conveniente porque permite alterações.

Porém, depois que o pipeline estiver funcionando corretamente, é interessante criar uma imagem `.sif` final.

Utilize:

```bash
apptainer build ambiente_final.sif env_machine_learning/
```

O resultado será:

```text
ambiente_final.sif
```

Esse arquivo representa uma versão congelada do ambiente.

## 9.1. Fluxo recomendado

```text
Ubuntu base
     │
     ▼
Criar Sandbox
     │
     ▼
Instalar ferramentas
     │
     ▼
Testar pipeline
     │
     ▼
Corrigir dependências
     │
     ▼
Pipeline funcionando
     │
     ▼
Gerar .sif
     │
     ▼
Usar imagem final
```

A partir desse ponto, você pode utilizar a imagem `.sif` para executar o pipeline de forma mais controlada e reprodutível.

---

# 10. Exemplo Completo de um Pipeline

Suponha que você tenha:

```text
/mnt/projeto_rnaseq/
├── dados/
│   └── matriz_expressao.csv
├── scripts/
│   └── analise.py
└── resultados/
```

E uma imagem:

```text
ambiente_final.sif
```

Você pode executar:

```bash
apptainer exec \
  --bind /mnt/projeto_rnaseq:/projeto \
  ambiente_final.sif \
  python3 /projeto/scripts/analise.py \
    --input /projeto/dados/matriz_expressao.csv \
    --output /projeto/resultados
```

Dentro do contêiner, o projeto será enxergado como:

```text
/projeto
├── dados/
│   └── matriz_expressao.csv
├── scripts/
│   └── analise.py
└── resultados/
```

Mas os arquivos continuam fisicamente armazenados no servidor.

---

# 11. Comandos Essenciais

| Objetivo | Comando |
|---|---|
| Ver versão | `apptainer --version` |
| Baixar imagem | `apptainer pull` |
| Criar sandbox | `apptainer build --sandbox` |
| Abrir contêiner | `apptainer shell` |
| Abrir sandbox para escrita | `apptainer shell --writable` |
| Executar comando | `apptainer exec` |
| Criar `.sif` | `apptainer build` |
| Montar diretório | `--bind` ou `-B` |
| Sair do contêiner | `exit` |

---

# 12. Boas Práticas para Projetos de Bioinformática

Para projetos de RNA-Seq, Machine Learning e outras análises bioinformáticas, uma organização simples pode ajudar bastante:

```text
projeto/
├── container/
│   └── ambiente_final.sif
│
├── scripts/
│   ├── preprocessamento.py
│   ├── analise.py
│   └── avaliacao.py
│
├── dados/
│   ├── matriz_tpm.csv
│   └── metadata.csv
│
├── resultados/
│   ├── tabelas/
│   ├── figuras/
│   └── modelos/
│
└── logs/
```

Uma boa estratégia é manter:

- o **código** fora do contêiner;
- os **dados** fora do contêiner;
- os **resultados** fora do contêiner;
- as **dependências e versões** dentro do contêiner.

Dessa forma, o `.sif` funciona como uma espécie de "fotografia" do ambiente computacional utilizado na análise.

---

# 13. Resumo do Fluxo de Trabalho

O fluxo recomendado pode ser resumido em cinco etapas:

### 1. Instalar

```bash
sudo apt update
sudo apt install -y apptainer
```

### 2. Configurar o cache

```bash
export APPTAINER_CACHEDIR=/caminho/para/cache
```

### 3. Desenvolver

```bash
apptainer build --sandbox meu_ambiente docker://ubuntu:22.04
```

Depois, instalar e testar as ferramentas necessárias.

### 4. Congelar

```bash
apptainer build ambiente_final.sif meu_ambiente/
```

### 5. Executar

```bash
apptainer exec \
  --bind /dados:/input \
  --bind /resultados:/output \
  ambiente_final.sif \
  python3 /input/script.py
```

---

# 14. Conceito Fundamental

A ideia central do Apptainer pode ser resumida assim:

> **O contêiner guarda o ambiente; o servidor guarda os dados.**

Ou seja:

```text
┌─────────────────────────────────────────┐
│              SERVIDOR                   │
│                                         │
│  Dados ───────────────┐                 │
│  Scripts ─────────────┤                 │
│  Resultados ◄─────────┤                 │
│                       │                 │
│                       ▼                 │
│              ┌─────────────────┐        │ 
│              │    APPTAINER    │        │ 
│              │                 │        │ 
│              │ Python / R      │        │
│              │ Bibliotecas     │        │
│              │ Ferramentas     │        │
│              └─────────────────┘        │
│                                         │
└─────────────────────────────────────────┘
```

Essa separação permite criar ambientes **reprodutíveis, isolados e compartilháveis**, sem precisar copiar os grandes volumes de dados para dentro do contêiner.

> **Documentação oficial:** https://apptainer.org/docs/user/latest/

