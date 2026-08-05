# LP-i10

Site institucional do **Instituto i10** (`www.institutoi10.com.br`): conjunto de **landing pages HTML estáticas** (FUNDEB por estado, campanhas regionais, APM, deputado360, etc.) hospedado na Vercel, que também atua como **hub de roteamento** — via `vercel.json` reescreve dezenas de rotas para outros apps i10 (CRM, marketing, insights, licitações, radar fiscal, propostas...). Há uma única função serverless com Neon.

## Stack
- **Linguagem:** HTML/CSS/JS estático (páginas independentes) + 1 função serverless em Node.js (ESM).
- **Backend:** `@neondatabase/serverless` (única dependência do `package.json`) usado em `api/anhumas.js`.
- **Banco:** Postgres (Neon), acessado apenas por `api/anhumas.js`.
- **Deploy:** **Vercel** (auto-deploy da branch `main`); domínio via `CNAME` → `www.institutoi10.com.br`.
- **Package manager:** npm (não há lockfile commitado; dependências mínimas).
- **Sem framework de build** — os arquivos `.html` são servidos como estão.

## Comandos
- **Não há scripts** em `package.json` (sem `dev`/`build`/`test`/`lint`).
- Preview local: servir a raiz como estático (ex.: `npx serve .`) ou usar `vercel dev` para exercitar também `api/anhumas.js` e as rewrites do `vercel.json`.
- Deploy: `git push origin main` (Vercel publica automaticamente).

## Estrutura
- `index.html` — home institucional.
- `vercel.json` — **coração do repo**: `rewrites` (rotas → páginas locais e proxies para apps externos `*.vercel.app`), `redirects` e `headers` (ex.: `no-cache` para `/apm-lp`).
- `CNAME` — domínio de produção.
- `api/anhumas.js` — função serverless (Neon) da campanha Anhumas.
- `fundeb-estado.html` — página única parametrizada por `?uf=xx`; o `vercel.json` mapeia `/fundeb-<uf>` para ela; `/fundeb-sp` e `/fundeb` têm páginas/hub próprios.
- Diretórios de campanha/landing: `anhumas/`, `apm-lp/`, `pa-smart/`, `marilia/`, `marilia-smart/`, `sorocaba/`, `sorocaba-smart/`, `regional/`, `deputado360/`, `estados/`, `fundeb/`, `fundeb-sp/`, `fundeb-reports/`, `sistemas/`.
- `assets/`, `images/`, `video/` — mídia estática.

## Convenções de código
- Cada landing é autocontida (HTML + CSS/JS inline ou local) — sem bundler, sem framework compartilhado.
- Ao adicionar uma página, registre a rota amigável correspondente em `vercel.json` (`rewrites`) e, se preciso, `redirects`.
- Rotas para outros sistemas são **proxies** para domínios `*.vercel.app` — não duplique o app aqui; ajuste apenas o destino.
- JS serverless (`api/`) em ESM; leia segredos sempre de `process.env`.

## Variáveis de ambiente
- **`DATABASE_URL`** — string de conexão Neon usada por `api/anhumas.js` (`neon(process.env.DATABASE_URL)`).
- Configurar no **painel da Vercel** (Project Settings → Environment Variables); em local, `.env*.local` (já no `.gitignore`). **Nunca** commitar a connection string.

## CI/CD & Deploy
- **Sem workflows** em `.github/workflows/`. O deploy é **Vercel auto-deploy da `main`**.
- Como não há build/typecheck, o risco de quebra é a sintaxe das páginas e, principalmente, o **JSON do `vercel.json`**.
- **CI mínimo recomendado (via PR)** em `.github/workflows/ci.yml`: validar `vercel.json` (ex.: `jq . vercel.json`) e rodar um HTML/link check nas páginas alteradas. Isso evita publicar rewrites malformadas que derrubam rotas em produção.

## Boas práticas de PR
- Branches: `feat/…`, `fix/…`, `chore/…`.
- Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`…).
- PRs pequenos. Checklist:
  - `vercel.json` é JSON válido e as novas rotas resolvem para o destino certo;
  - nenhum segredo/`.env` no diff;
  - **nenhuma lista de destinatários / CSV com PII** (o `.gitignore` já bloqueia `**/destinatarios-*.csv` — mantenha assim);
  - screenshots de antes/depois para mudanças de UI;
  - preview da Vercel conferido antes do merge.
- Pelo menos 1 review; squash merge; `main` sempre deployável (é produção direta).

## Testes
- Não há testes (site estático).
- Proporcional: um lint de HTML/links e validação do `vercel.json` no CI cobrem os erros mais comuns. Para `api/anhumas.js`, um smoke test manual via `vercel dev` antes do merge.

## Segurança & dados
- **Nunca** commitar `.env`, `DATABASE_URL` ou listas de e-mail/telefone (PII → **LGPD**). O site é público e estático: qualquer coisa commitada fica exposta.
- O `.gitignore` já protege `destinatarios-*.csv`, `.env*.local`, `node_modules/` e `.vercel` — não afrouxe.
- Rewrites com `no-cache` (ex.: `/apm-lp`) existem de propósito; ao adicionar campanhas voláteis, avalie cache.
- Revisar a dependência Neon periodicamente.

## Gotchas
- **`vercel.json` é o ponto mais frágil:** um erro de JSON ou uma rota fora de ordem/errada quebra navegação em produção. A ordem das regras importa (a primeira que casa vence) — cuidado com prefixos que capturam rotas mais específicas.
- Muitas rotas apontam para **outros repositórios/deploys** (`ldo-dados-sp`, `i10-marketing`, `i10-insights`, `bncc-captacao`, `i10-licitacoes-360`, `agentes-i10`, etc.): alterações de path nesses apps podem quebrar o proxy aqui, e vice-versa.
- `/fundeb-<uf>` compartilha **uma única** `fundeb-estado.html` via query `?uf=`; mudanças nessa página afetam todos os estados de uma vez.
- Deploy é direto na `main` = produção. Não faça commits experimentais na `main`; use branch + preview.
- Sem build: erros só aparecem no navegador/produção — teste com `vercel dev` ou preview.
