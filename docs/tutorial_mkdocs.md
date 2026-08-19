# Como criar um site de documentação com MkDocs e publicar no GitHub Pages

Neste tutorial, você vai aprender a transformar arquivos Markdown (`.md`) em um site interativo e profissional usando o **MkDocs** (com o tema *Material for MkDocs*) e hospedá-lo gratuitamente no **GitHub Pages**.

Esta é uma excelente forma de documentar pipelines de análise de dados, scripts e criar um portfólio acessível.

## 1. O que é o MkDocs?

O MkDocs é um gerador de sites estáticos rápido e simples, construído em Python. Ele pega arquivos de texto escritos em Markdown e os transforma em um site completo. O tema **Material for MkDocs** adiciona um visual moderno, abas interativas, botões de cópia de código e suporte a modo claro/escuro.

## 2. Instalação

Você precisará do Python instalado. Abra o terminal e instale o pacote via `pip`:

```bash
pip install mkdocs-material
```

## 3. Inicializando o Projeto

Vá até a pasta onde deseja criar seu site e execute:

```bash
mkdocs new meus-tutoriais
cd meus-tutoriais
```

Isso criará a seguinte estrutura:
```text
meus-tutoriais/
├── mkdocs.yml    # Arquivo principal de configuração
└── docs/
    └── index.md  # Sua página inicial
```

## 4. Personalizando o Site (`mkdocs.yml`)

Para deixar o site com a sua identidade visual (incluindo logo, paleta de cores e funcionalidades extras), edite o arquivo `mkdocs.yml`. Exemplo de configuração:

```yaml
site_name: Tutoriais e Scripts - Márcio Wilson
site_description: Repositório de tutoriais, scripts de bioinformática e documentações
theme:
  name: material
  logo: assets/logo.png      # Coloque sua logo (fundo transparente) em docs/assets/
  favicon: assets/logo.png
  features:
    - navigation.tabs        # Menus em abas
    - content.code.copy      # Botão de copiar código
  palette:
    - scheme: default
      primary: teal          # Cor principal
      accent: cyan
      toggle:
        icon: material/brightness-7 
        name: Mudar para modo escuro
    - scheme: slate
      primary: teal
      accent: cyan
      toggle:
        icon: material/brightness-4
        name: Mudar para modo claro

markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences
  - pymdownx.highlight:
      anchor_linenums: true
      line_spans: __span

nav:
  - Início: index.md
  - Ferramentas e Infra:
    - Singularity (Apptainer): tutorial_singularity.md
    - MkDocs e GitHub Pages: tutorial_mkdocs.md
```

## 5. Testando Localmente

Antes de publicar, veja como o site está ficando na sua própria máquina. Execute:

```bash
mkdocs serve
```
Abra o link (geralmente `http://127.0.0.1:8000`) no seu navegador. O site será atualizado automaticamente sempre que você salvar um arquivo `.md`.

## 6. Publicando no GitHub Pages (Gratuito)

O GitHub Pages permite hospedar sites estáticos de repositórios públicos gratuitamente. A grande vantagem é que o MkDocs automatiza o processo de publicação com um único comando.

**Passo A:** Inicie o repositório Git e conecte ao GitHub (lembre-se de criar um repositório vazio no GitHub primeiro):
```bash
git init
git add .
git commit -m "Site inicializado"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
git push -u origin main
```

**Passo B:** Construa e publique o site:
```bash
mkdocs gh-deploy
```

Pronto! O MkDocs vai compilar os arquivos e enviá-los para uma branch dedicada chamada `gh-pages`. Em poucos minutos, seu site estará online no endereço `https://SEU_USUARIO.github.io/SEU_REPOSITORIO/`. 

Sempre que adicionar um novo tutorial (ou criar novas documentações para pipelines como RNA-Seq, por exemplo), basta rodar `mkdocs gh-deploy` novamente.