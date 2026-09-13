# ms-v7 — Social Autopilot

Um arquivo de voz de marca + 3 prompts → posts nativos por rede, na sua voz, agendados pelo provedor que você escolher. **Nada vai ao ar sem o seu "sim".**

Adaptado de *The Social Autopilot* (Zubair Trabzada, AI Workshop). Os originais (prompt pack, template de voz, transcrição do vídeo) estão em `docs/`. O plano de implementação está em `docs/PLANO.md`.

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/ms-v7/guia/**

---

## 1. Como funciona (em 1 minuto)

```
brand-voice.md  ─┐
config.yaml     ─┼─►  /ms-post  ──► 3 perguntas ──► 3 opções ──► visual ──► "Agendo? (sim/rascunho/não)"
provedores      ─┘        │
                          ├─► /ms-semana       retro + tabela de 5-7 posts + fila da semana
                          └─► /ms-reaproveita  transcrição / artigo / URL → um post por rede
```

- **Voz de marca** (`brands/<slug>/brand-voice.md`): quem você é, público, ofertas, frases, histórias verdadeiras, regras por rede. É o que faz o post soar como você e não como IA.
- **Config** (`config.yaml`): qual marca está ativa, quais redes recebem posts, quem publica (Metricool, Blotato ou copiar à mão), quem gera imagem/vídeo.
- **Skills** (`.claude/skills/`): os prompts do pack empacotados como comandos do Claude Code.
- **Pipelines** (`pipeline/`): receitas passo a passo de cada provedor.

---

## 2. Start (pré-requisitos)

| O quê | Como | Obrigatório? |
|---|---|---|
| Claude Code | `claude.com/claude-code` — plano Pro ou superior | sim |
| Este repo | `git clone git@github.com:inematds/ms-v7.git ~/projetos/ms-v7` | sim |
| Python 3 + PyYAML | `pip install pyyaml` (só para validar o `config.yaml`) | recomendado |
| Provedor de publicação | Metricool (MCP, grátis) ou Blotato (MCP, pago) — ver §4 | não (sem ele, modo `copiar`) |
| Gerador de imagem | flux2-klein local (padrão) ou Magnific (MCP) | não |
| Host de mídia | `gh` autenticado na conta `inematds` (release em `inematds/midia`) | só com provedor MCP |

Abra o Claude Code **dentro da pasta**:

```bash
cd ~/projetos/ms-v7
claude
```

Ele lê `CLAUDE.md` automaticamente e passa a conhecer as regras e os comandos.

---

## 3. Configurar a primeira marca

No Claude Code:

```
/ms-config
```

O comando pergunta **uma coisa por vez**, em texto livre. Ele faz três blocos:

**A. Voz de marca** → cria `brands/<slug>/brand-voice.md` a partir de `brands/_template/`.
- Se a marca já está descrita em algum lugar (uma skill, um site, um doc), diga onde: ele pré-preenche e só pergunta o que falta.
- O que ele sempre pergunta porque ninguém pode inventar: 5 frases que você realmente fala, 3 histórias verdadeiras com número, prova verificável, CTAs exatos.
- Respostas vagas são recusadas ("muitos alunos" → "quantos?").
- Termina com um **teste de voz**: ele escreve uma legenda só com o arquivo e pergunta "você escreveria isso?".

**B. Redes e provedores** → preenche `config.yaml`.
- Para cada rede: handle e se está `ativa`. Só redes ativas recebem posts.
- Provedor de publicação: `metricool` · `blotato` · `copiar`.
- Imagem: `flux2-klein` (padrão) ou `magnific`. Vídeo: `nenhum` até precisar.

**C. Ajustes depois**: `"ativa o LinkedIn"`, `"troca pra blotato"`, `"usa a marca X"` — ele edita só a chave pedida.

Também dá pra editar `config.yaml` à mão. Validar:

```bash
python3 -c "import yaml; yaml.safe_load(open('config.yaml'))" && echo ok
```

### Várias marcas

Cada marca é uma pasta em `brands/`. Só uma é a ativa (`marca_ativa` no `config.yaml`). Para trocar: `"usa a marca X"` ou edite a chave.

---

## 4. Conectar um provedor de publicação

### Metricool (validado, plano Free)

```bash
claude mcp add --transport http metricool https://ai.metricool.com/mcp -s user
```

Dentro do Claude Code: `/mcp` → **metricool** → **Authenticate** → autorizar no navegador. Depois, no `/ms-config`, informe o `blogId` (ele descobre com `getBrandSettings`).

Limites do Free: 1 marca, 1 conta por rede, **sem LinkedIn nem X**, **20 posts/mês**, analytics de 30 dias, sem "apagar post" pelo MCP (cancela no app).

### Blotato (não testado aqui)

Criar conta → conectar redes → copiar a URL do MCP → `claude mcp add --transport http blotato <url> -s user`. Antes de usar em produção, faça um post em `rascunho` e anote em `pipeline/publicar.md` o payload que funcionou.

### Copiar (sem API)

Nada a conectar. As skills entregam o texto pronto em bloco de código e você cola na rede. É o padrão enquanto nenhum provedor estiver configurado.

### Host de mídia

Provedores MCP só aceitam **URL direta** de imagem/vídeo. O padrão é um GitHub Release:

```bash
gh release upload v1 arquivo.png --repo inematds/midia --clobber
# → https://github.com/inematds/midia/releases/download/v1/arquivo.png
```

As skills fazem isso sozinhas; só precisa do `gh` logado na conta `inematds`.

---

## 5. Uso diário

| Comando | O que acontece | Tempo |
|---|---|---|
| `/ms-post` | 3 perguntas (rede, tema, objetivo) → pesquisa opcional → 3 opções → você escolhe → visual → "Agendo?" | 3-5 min |
| `/ms-semana` | Retro da semana passada (analytics) → 3 perguntas → tabela de 5-7 posts → você edita/aprova → escreve tudo → visuais → "Agendo a fila?" | 15 min |
| `/ms-reaproveita` | Cola transcrição/artigo/URL → "quais redes?" → um post autônomo por rede → fila | 5 min |

**O gate de aprovação** aceita três respostas:

- `sim` → agenda com publicação automática no horário proposto.
- `rascunho` → cria como rascunho no provedor (não publica).
- `não` → só o texto, pra copiar.

Cada post fica em `posts/<data>-<slug>/post.md` com status, `id`/`uuid` do provedor e o visual aprovado. Semanas ficam em `posts/semanas/`.

---

## 6. Publicar no git

O trabalho **termina no push**. Deploy é responsabilidade do webhook, não sua.

```bash
cd ~/projetos/ms-v7
git config user.email   # tem que ser inematds@gmail.com
git add -A
git commit -m "post: 2026-09-15-ler-documentacao (instagram, agendado)"
git push
```

Regras:
- Repo: `inematds/ms-v7`. Autor e committer: `inematds <inematds@gmail.com>`. Se `git config user.email` divergir, corrija **localmente** (`git config user.email inematds@gmail.com`), sem tocar a config global.
- As skills já commitam ao final de cada post/semana. Você só dá o `git push`.
- Vídeos e imagens fora de `posts/` são ignorados pelo `.gitignore`. Mídia pública vai em release no repo `inematds/midia`, nunca no histórico deste.
- Versão em `VERSION` e no topo de `CLAUDE.md` (`vX.XX.YY`: patch incrementa `YY`; feature incrementa `XX` e carrega `YY`; só major zera).
- Falha real → uma linha em `FALHAS.md` (`| data | o que quebrou | menor correção | prompt \| infra |`).

---

## 7. Publicar no portal (inema.club)

O portal aponta para uma **página landing + guia** servida pelo GitHub Pages **deste mesmo repo** (nunca um repo separado).

**Passo 1 — criar o guia** (já existe em `guia/index.html`; regerar se mudar o projeto):

```
/projetos-landing-guia
```

Gera `guia/index.html` (self-contained, padrão INEMA dark âmbar) com `guia/assets/` ao lado.

**Passo 2 — ativar o GitHub Pages via GitHub Actions** (o workflow `.github/workflows/pages.yml` já está no repo; não usar o build "legacy" por branch, que trava):

```bash
gh repo edit inematds/ms-v7 --visibility public --accept-visibility-change-consequences   # Pages exige repo público no plano free
gh api -X POST repos/inematds/ms-v7/pages -f build_type=workflow \
  || gh api -X PUT repos/inematds/ms-v7/pages -f build_type=workflow
git add guia && git commit -m "docs: guia landing" && git push   # o push dispara o deploy
gh run list --repo inematds/ms-v7 --limit 3                       # acompanhar
```

> Commits que tocam `.github/workflows/` precisam ir por SSH (`git@github.com:inematds/ms-v7.git`): o token HTTPS da conta `inematds` não tem o escopo `workflow`. O remote deste repo já é SSH.

URL resultante: `https://inematds.github.io/ms-v7/guia/`. Conferir com `curl -sI <url> | head -1` (tem que responder 200).

> Antes de tornar público: `brands/` contém sua voz de marca e histórias pessoais, e `posts/` contém o histórico. Se não quiser isso público, mova o guia para um branch `gh-pages` só com `guia/`, ou mantenha o repo privado e hospede o guia em outro lugar.

**Passo 3 — colocar no portal:**

```
/atualiza-portal https://inematds.github.io/ms-v7/guia/
```

A skill cria o card nas 3 superfícies INEMA (portal `inema.club`, inemabuscas e catálogo PRO), commita e faz push nos 3 repos. O deploy no Vercel é automático pelo webhook; não precisa (nem deve) checar o dashboard.

---

## 8. Mapa do repo

```
ms-v7/
├── CLAUDE.md                  regras fixas + comandos (o Claude Code lê sozinho)
├── README.md                  este arquivo
├── VERSION                    1.0.0
├── FALHAS.md                  changelog de falhas, uma linha cada
├── config.yaml                marca ativa · redes · provedores · regras
├── brands/
│   ├── _template/brand-voice.md   template de voz (PT-BR, todas as redes)
│   └── <slug>/brand-voice.md      uma pasta por marca
├── pipeline/
│   ├── publicar.md            Metricool (validado) · Blotato · copiar · hospedagem
│   └── visual.md              formatos por rede · flux2-klein · Magnific · vídeo
├── posts/
│   ├── <AAAA-MM-DD>-<slug>/post.md   cada post, com status e id/uuid
│   └── semanas/<AAAA-WW>.md          plano + retro de cada semana
├── .claude/skills/
│   ├── ms-config/   ms-post/   ms-semana/   ms-reaproveita/
├── guia/                      landing + guia (GitHub Pages) → inematds.github.io/ms-v7/guia/
└── docs/                      originais do pack + PLANO.md
```

---

## 9. Problemas comuns

| Sintoma | Causa provável | Fix |
|---|---|---|
| "nenhuma marca configurada" | `marca_ativa: null` | `/ms-config` |
| Skill entrega só texto, não agenda | provedor = `copiar` ou MCP desconectado | `/mcp` → Authenticate, ou `/ms-config` "troca pra metricool" |
| `createScheduledPost` recusa a mídia | URL não é arquivo direto ou não responde 200 | `curl -sI <url>`; refazer o `gh release upload` |
| Post duplicou no YouTube | rede `youtube` ativa numa marca cujo canal é origem de outro pipeline | `ativa: false` para youtube no `config.yaml` |
| Analytics vazio na retro | janela de 30 dias do Free, ou conector errado | ver `pipeline/publicar.md` § Analytics |
| Push recusado por autor | `user.email` errado | `git config user.email inematds@gmail.com` e commit vazio |
