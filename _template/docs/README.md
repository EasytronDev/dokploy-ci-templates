# _template — Como replicar pra um novo projeto

> **Use este README quando quiser adicionar CI/CD em outro projeto.**
> Cada passo e copy-paste friendly. Tempo total: **~15 minutos**.

---

## Pre-requisitos

Antes de comecar, tenha em mao:

- [ ] Acesso ao GitHub (proprietario ou write no repo do projeto)
- [ ] Acesso ao Dokploy UI (https://dokploy.easytron.com.br) como admin
- [ ] Permissao pra criar Secrets no GitHub repo

> Se voce NAO tem algum desses, peca ao admin do time.

---

## Conceito rapido (1 min de leitura)

O CI/CD faz 4 coisas por PR:

1. **Cria app `pr-N` no Dokploy** (onde N = numero do PR)
2. **Configura build source** (GitHub repo + branch + Nixpacks)
3. **Configura envs** (de `CONSTANTINE_*` secrets do GitHub)
4. **Faz deploy** e mostra a URL `pr-N.dokploy.easytron.com.br`

Quando a PR fecha, **deleta o app** (cleanup).

**Voce nao precisa entender tudo isso.** So seguir os passos.

---

## Passo 0: Clonar o repo de templates

```bash
# Clone o repo publico com os templates
git clone https://github.com/EasytronDev/dokploy-ci-templates.git
cd dokploy-ci-templates

# Estrutura:
ls
# README.md
# _template/           <-- voce usa este
# constantine/         <-- referencia aplicada
```

---

## Passo 1: Copiar o template pro seu projeto

```bash
# Copia a pasta _template com nome do seu projeto
cp -r _template/ meu-novo-projeto/

# Verifica
ls meu-novo-projeto/
# README.md (este arquivo)
# workflows/
#   preview-deploy.yml
#   preview-cleanup.yml
```

---

## Passo 2: Substituir placeholders

Edite os arquivos em `meu-novo-projeto/workflows/`. Vou usar como exemplo o projeto **EasytronDev/Constantine**:

| Placeholder | O que e | Exemplo (Constantine) |
|-------------|--------|----------------------|
| `REPO_OWNER` | Owner do repo GitHub | `EasytronDev` |
| `REPO_NAME` | Nome do repo | `Constantine` |
| `BUILD_PATH` | Path do app dentro do repo | `/services/frontend/apps/web/` |
| `DEFAULT_PORT` | Porta que o app escuta | `3000` |
| `MEMORY_LIMIT` | Limite de memoria (MB) | `512` |
| `MEMORY_RESERVATION` | Reserva de memoria (MB) | `128` |

**Substituicao automatica com `sed`:**

```bash
cd meu-novo-projeto/workflows

# Edite essas 6 linhas com os valores do SEU projeto:
sed -i 's/REPO_OWNER/EasytronDev/g;          \
         s/REPO_NAME/Constantine/g;           \
         s|BUILD_PATH|/services/frontend/apps/web/|g; \
         s/DEFAULT_PORT/3000/g;               \
         s/MEMORY_LIMIT/512/g;                 \
         s/MEMORY_RESERVATION/128/g'           \
    preview-deploy.yml preview-cleanup.yml

# Verificar (deve listar ZERO matches)
grep -E 'REPO_OWNER|REPO_NAME|BUILD_PATH|DEFAULT_PORT|MEMORY' preview-deploy.yml preview-cleanup.yml
```

> **Se aparecer matches, voce errou a substituicao.** Reveja o `sed` acima.

---

## Passo 3: Criar projeto no Dokploy

1. Abre https://dokploy.easytron.com.br
2. Login
3. **Projects** > **Create** (botao no canto superior direito)
4. Preencher:
   - **Name:** `meu-novo-projeto` (mesmo nome do repo ou outro)
   - **Description:** descricao do projeto
5. Click **Create**

**Anotar o ID do projeto:**

Apos criar, voce sera redirecionado pra pagina do projeto. **OLHE A URL**:

```
https://dokploy.easytron.com.br/project/k-AbCdEfGhIjK
                              ^^^^^^^^^^^^^^^^
                              ESTE E O SEU PROJECT ID
```

**Anote:** `DOKPLOY_PROJECT_ID = k-AbCdEfGhIjK`

---

## Passo 4: Pegar o Environment ID

1. Ainda na pagina do projeto, click em **Production** (environment padrao)
2. OLHE A URL:

```
https://dokploy.easytron.com.br/project/k-AbCdEfGhIjK/m/mNjKlMnOpQrSt
                                               ^^^^^^^^^^^^^^^^
                                               ESTE E O SEU ENV ID
```

**Anote:** `DOKPLOY_ENV_ID = mNjKlMnOpQrSt`

> Se voce usa environment diferente de "production", pegue o ID dele.

---

## Passo 5: Configurar GitHub Provider no Dokploy

> **Se ja existe um GitHub Provider** (caso tenha outro projeto ja configurado), **pule este passo** e reusa o `githubId`.

1. **Settings** > **Git Providers** (no menu lateral)
2. **Create** > **GitHub**
3. Preencher:
   - **Name:** `meu-novo-projeto-github` (ou qualquer nome)
   - **GitHub App:** seguir wizard do Dokploy
4. Click **Create**

**Pegar o githubId:**

Apos criar, abra **terminal local** (ou use o Swagger UI em https://dokploy.easytron.com.br/swagger):

1. Va em **Settings** > **Profile** > **API/CLI**
2. Crie um token (se nao tiver) e copie
3. Va no **Swagger UI** (https://dokploy.easytron.com.br/swagger)
4. Click no cadeado "Authorize" e cole o token
5. Procure `gitProvider.getAll` > Try it out > Execute
6. Copie o `githubId` do seu provider (formato: 22 chars tipo `xipBaI2_njHdFq_b8jv-6`)

**Anote:** `DOKPLOY_GITHUB_ID = xipBaI2_njHdFq_b8jv-6`

---

## Passo 6: Pegar DOKPLOY_API_KEY

1. **Settings** > **Profile** > **API/CLI**
2. Se nao tem token, click **Create Token**
3. Copie o token (formato: 75 chars tipo `constantineqSprlqdkaXVSXGvXiyXYf...`)

**Anote:** `DOKPLOY_API_KEY=consta...n
> Trate como SENHA. Nao commite em lugar nenhum.

---

## Passo 7: Adicionar 5 secrets no GitHub do novo projeto

1. Va em `https://github.com/REPO_OWNER/REPO_NAME/settings/secrets/actions/new`
2. Click **New repository secret** e adicione UM por UM:

| Name | Value |
|------|-------|
| `DOKPLOY_URL` | `https://dokploy.easytron.com.br` |
| `DOKPLOY_API_KEY` | (o token do Passo 6) |
| `DOKPLOY_PROJECT_ID` | (o ID do Passo 3) |
| `DOKPLOY_ENV_ID` | (o ID do Passo 4) |
| `DOKPLOY_GITHUB_ID` | (o ID do Passo 5) |

> **Dica importante:** ao colar `DOKPLOY_API_KEY` no campo, **digite manualmente**
> se o `sed`/colar adicionar newline/space no final. Esses chars invisiveis
> quebram o shell no GitHub Actions.

---

## Passo 8: Copiar workflows pro repo

```bash
# Va ate o repo do seu projeto
cd /path/to/seu-projeto

# Cria pasta de workflows
mkdir -p .github/workflows

# Copia os 2 workflows (ja com placeholders substituidos)
cp /caminho/para/meu-novo-projeto/workflows/*.yml .github/workflows/

# Verifica o conteudo
cat .github/workflows/preview-deploy.yml | head -30
# Deve mostrar SEU_REPO_OWNER, SEU_REPO_NAME, etc — NAO os placeholders

# Commit + push
git add .github/workflows/
git commit -m "ci: add Dokploy preview workflows"
git push origin develop
```

---

## Passo 9: Testar com PR de verdade

```bash
# Cria branch de teste
git checkout -b feature/test-cicd
echo "test" > test.txt
git add test.txt
git commit -m "test: verify CI/CD"
git push origin feature/test-cicd

# Cria PR via GitHub UI:
#   https://github.com/REPO_OWNER/REPO_NAME/compare/develop...feature/test-cicd
#   Title: "Test CI/CD"
#   Base: develop
```

---

## Passo 10: Validar que funcionou

1. Vai em `https://github.com/REPO_OWNER/REPO_NAME/actions`
2. Click no run "Preview Deploy" (deve estar rodando)
3. Aguarda ficar **verde** (5-10min)
4. **Se vermelho**, veja `../constantine/docs/troubleshooting.md`

Quando verde:
1. Vai em https://dokploy.easytron.com.br
2. **Projects** > `meu-novo-projeto` > **Production**
3. Deve aparecer app `pr-1` (ou numero do PR)
4. Click no app > aba **General**:
   - **GitHub Account:** preenchido
   - **Repository:** `REPO_NAME`
   - **Branch:** `feature/test-cicd`
5. Click na aba **Domains**:
   - Deve ter `pr-1.dokploy.easytron.com.br`

**Se tudo isso estiver OK, CI/CD funcionando!**

---

## Validacao final

Acessa `https://pr-1.dokploy.easytron.com.br` no browser. **Deve aparecer a app.**

Se nao aparecer:
- Espera 5-10min (build do Nixpacks)
- Click no app > **Logs** pra ver se tem erro
- Veja `../constantine/docs/troubleshooting.md`

---

## Customizacoes por projeto

| Caso | Ajuste |
|------|--------|
| Static site (Next.js export, Astro, Hugo) | `buildType: "static"` no `preview-deploy.yml` |
| Python (Django, Flask) | `buildType: "python"` |
| Multi-port (ex: 3000 + 8080) | Adicionar 2 `Add domain` steps |
| Banco de dados | Adicionar envs com connection string no `Configure envs` |
| Docker custom (Dockerfile proprio) | Trocar `buildType: "nixpacks"` por `"dockerfile"` |

---

## Se der problema

1. **Log do Actions:** `https://github.com/REPO_OWNER/REPO_NAME/actions` > run failed > step que falhou
2. **Troubleshooting completo:** `https://github.com/EasytronDev/dokploy-ci-templates/blob/main/constantine/docs/troubleshooting.md`
3. **Doc do projeto (exemplo aplicado):** `https://github.com/EasytronDev/dokploy-ci-templates/blob/main/constantine/docs/setup-guide.md`

---

## Estrutura final do repo do SEU projeto

```
seu-projeto/
└── .github/
    └── workflows/
        ├── preview-deploy.yml    # Cria/deploya preview por PR
        └── preview-cleanup.yml   # Deleta preview quando PR fecha
```

**Esses 2 arquivos sao TUDO que voce precisa adicionar.**

---

## TL;DR (pra quem tem pressa)

```bash
# 1. Clone templates
git clone https://github.com/EasytronDev/dokploy-ci-templates.git
cd dokploy-ci-templates

# 2. Copia + substitui placeholders
cp -r _template/ meu-app/
cd meu-app/workflows
sed -i 's/REPO_OWNER/SuaOrg/g; s/REPO_NAME/seu-app/g; \
         s|BUILD_PATH|/|g; s/DEFAULT_PORT/3000/g; \
         s/MEMORY_LIMIT/512/g; s/MEMORY_RESERVATION/128/g' \
    preview-*.yml

# 3. Cria projeto no Dokploy UI, anota IDs

# 4. Adiciona 5 secrets no GitHub repo

# 5. Copia workflows
cd /path/to/seu-app
mkdir -p .github/workflows
cp /caminho/meu-app/workflows/*.yml .github/workflows/
git add .github/workflows/ && git commit -m "ci: add" && git push

# 6. Cria PR de teste, valida URL
```

**Total: 15 minutos do zero ao PR preview funcionando.**
