# Como criar um site de documentação com MkDocs e publicar no GitHub Pages

O MkDocs é um gerador de sites estáticos. Ele converte textos simples escritos em Markdown (`.md`) num site interativo, rápido e profissional. Combinado com o tema **Material for MkDocs**, ele oferece botões de cópia, modo escuro e caixas de alerta.

## 1. Instalação e Criação do Projeto

No seu terminal, instale a ferramenta usando o Python:
```bash
pip install mkdocs-material
```

Crie a estrutura base do site:
```bash
mkdocs new meus-tutoriais
cd meus-tutoriais
```
Isso gera o arquivo `mkdocs.yml` (configurações gerais) e a pasta `docs/` com seu `index.md`.

## 2. Testando Localmente

Para ver as mudanças em tempo real enquanto edita os textos, rode:
```bash
mkdocs serve
```
Acesse `http://127.0.0.1:8000` no seu navegador. O site recarrega automaticamente ao salvar os arquivos.

## 3. Como Publicar no GitHub Pages (Passo a Passo)

O GitHub Pages é um serviço gratuito do GitHub que hospeda sites estáticos a partir de repositórios. O MkDocs tem um atalho nativo para automatizar toda a publicação.

### Etapa A: Criar o repositório no site do GitHub
Para que o site fique online, ele precisa morar em um repositório no GitHub.

1. Acesse o site do [GitHub](https://github.com/) e faça login.
2. No canto superior direito, clique no sinal de **`+`** e escolha **New repository**.
3. Em **Repository name**, dê um nome curto (ex: `tutoriais-scripts`).
4. Em **Public/Private**, certifique-se de deixar como **Public** (o Pages gratuito exige repositórios públicos).
5. **Muito Importante:** Deixe **DESMARCADAS** as opções *Add a README file*, *Add .gitignore* e *Choose a license*. O repositório precisa nascer completamente vazio.
6. Clique no botão verde **Create repository**.

O GitHub mostrará uma página cheia de códigos. Copie a URL do seu repositório (ex: `https://github.com/SeuUsuario/tutoriais-scripts.git`).

### Etapa B: Conectar a pasta do seu computador ao GitHub
Abra o seu terminal na pasta do seu site (a pasta `meus-tutoriais` onde está o `mkdocs.yml`) e rode estes comandos em ordem:

```bash
# 1. Transforma sua pasta num repositório Git local
git init

# 2. Adiciona todos os seus arquivos (Markdown, logo, mkdocs.yml)
git add .

# 3. Salva uma "fotografia" do estado atual dos arquivos
git commit -m "Meu primeiro site estruturado"

# 4. Renomeia a branch principal para 'main'
git branch -M main

# 5. Conecta sua pasta local ao repositório vazio que você acabou de criar no site
# Lembre-se de substituir pela URL que você copiou no Passo A!
git remote add origin https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git

# 6. Envia os arquivos "crus" (.md, .yml) para o GitHub
git push -u origin main
```

### Etapa C: A Mágica do MkDocs (Colocando online!)
Até agora, você enviou os textos crus para o GitHub. Para transformar isso em um site de verdade e publicá-lo, basta rodar **um único comando**:

```bash
mkdocs gh-deploy
```

**O que esse comando faz?**
Ele lê seus arquivos Markdown, compila todo o HTML/CSS do site e cria automaticamente uma branch oculta no seu repositório chamada `gh-pages`. O GitHub percebe essa branch e coloca o site no ar instantaneamente.

**Como acessar o site?**
O endereço será sempre neste formato:
`https://SEU_USUARIO.github.io/NOME_DO_REPOSITORIO/`

*Nota: Na primeira vez, o GitHub pode levar até 2 minutinhos para terminar de processar o link. É só recarregar a página.*

---

**Sempre que quiser atualizar o site:**
1. Edite ou adicione novos arquivos `.md` na pasta `docs/`.
2. Adicione os novos arquivos no índice `nav:` dentro do `mkdocs.yml`.
3. Abra o terminal e rode `mkdocs gh-deploy` novamente. Simples assim!