# MS-v7 — Social Autopilot INEMA · Plano de Implementação

> **Para agentes executores:** usar `superpowers:executing-plans` (inline) ou `superpowers:subagent-driven-development`, tarefa por tarefa. Passos usam checkbox (`- [ ]`). Perguntas ao usuário são SEMPRE em texto livre (nunca `AskUserQuestion`).

> **Nota de execução (2026-09-13):** o projeto foi construído com **provedores, marca e redes configuráveis** em `config.yaml` (decisão do usuário), em vez de fixar Metricool/inemafuturos. As Tasks 0, 2, 3, 4 e 5 estão implementadas como skills + receitas genéricas; a Task 1 (voz de marca) virou `/ms-config` e roda quando o usuário configurar a primeira marca. As "Perguntas em aberto" passam a ser respondidas dentro do `/ms-config`.

**Objetivo:** Reproduzir o "Social Autopilot" (Zubair Trabzada) na stack do Nei: um `brand-voice.md` da INEMA + 3 prompts (diário, semanal, reaproveitar) que pesquisam, escrevem no tom da marca, geram o visual, e **agendam pelo Metricool só depois do "sim"** explícito.

**Arquitetura:** Zero código de aplicação. O sistema é (1) um arquivo de voz, (2) três skills locais do projeto que encapsulam os prompts do pack, (3) o pipeline visual → host público → `createScheduledPost`. Tudo roda dentro do Claude Code neste diretório.

**Stack:** Claude Code (Fable 5.1) · Metricool MCP (já conectado, OAuth) · flux2-klein (imagem, default) / Magnific (fallback) · `gh release upload` em `inematds/midia` (host de mídia) · git → `inematds/ms-v7`.

**Spec:** `docs/Social-Autopilot-Prompt-Pack.txt` (os 3 prompts + bônus), `docs/brand-voice-template.md` (template), `docs/social-media-automation-fable-5.md` (transcrição do vídeo). Referência da integração já validada: `~/projetos/pubmetricool/docs/`.

## Substituições em relação ao vídeo

| No vídeo | Aqui | Por quê |
|---|---|---|
| Blotato MCP (posta) | **Metricool MCP** | Já conectado e validado em 2026-08-19 (`pubmetricool`). Free, permanente. |
| Higgsfield MCP (imagem/vídeo) | **flux2-klein** por default; **Magnific** quando o flux não der conta (referência de personagem, vídeo) | Regra global do CLAUDE.md. |
| Upload direto de mídia pelo MCP | **GitHub Release em `inematds/midia`** → URL pública | Metricool só aceita URL direta de arquivo. Passo que o pack não tem. |
| 6 plataformas (IG, X, LinkedIn, TikTok, FB, YT) | **Instagram + TikTok** na fase 1 | Plano Free da Metricool não conecta LinkedIn nem X. YouTube conectado é o canal-origem do `yt-pub-lives` (agendar lá duplica). |
| Brand voice genérico (Alex, café) | **INEMA / Nei**, semeado da skill `roteirista-inema` | Contexto fixo já existe: 40 anos em tecnologia, anti-guru, só CLUB em Reels, público 40+. |
| "Routines" do Claude Desktop | Skill `/ms-semana` rodada à mão (15 min/semana) | Sem daemon; aprovação humana é feature. |

## Restrições globais

- Marca Metricool: `label inemafuturos · blogId 6745962 · timezone America/Sao_Paulo`. Redes: Instagram `@inemafuturos`, TikTok `@lnema.club`, YouTube `UC2QbQDyPKuHk93dwo5iq3Sw` (**não agendar** YouTube — ver Perguntas em aberto).
- Cota: **20 posts/mês** no Free (não documentado se `draft` ou post multi-rede conta 1 ou N — conferir no app). Analytics: janela de **30 dias**. Não há ferramenta de *delete* no MCP → cada teste real gasta cota e precisa ser cancelado no app.
- `createScheduledPost` exige `publicationDate`; "agora" = agora + 5 min com `autoPublish: true`. `updateScheduledPost` exige `id` **e** `uuid` e o payload completo.
- Mídia: Reels/TikTok 9:16 (1080×1920); feed IG 4:5 (1080×1350); carrossel IG até 10 imagens 4:5.
- Nunca publicar/agendar sem "sim" explícito do usuário no chat. Nunca inventar números, histórias ou estatísticas.
- Palavras proibidas (herdadas do roteirista-inema): "segredo", "hack", "truque", "fórmula mágica", "método infalível", "renda extra". Autoridade: **"40 anos em tecnologia"**, nunca "40 anos em IA". Em Reels só se menciona o INEMA.CLUB (nunca PRO/imersões).
- Anti-AI-speak (do pack): sem "game-changer", "unlock", "delve", "elevate", pergunta retórica de abertura, muro de hashtags.
- Git: repo `inematds/ms-v7`, autor `inematds <inematds@gmail.com>`. Publicar = commit + push.
- Versão: `v1.0.0` ao fechar a Fase 1. Semver do CLAUDE.md global.

## Estrutura de arquivos

```
ms-v7/
├── CLAUDE.md                     # contexto do projeto p/ o Claude Code (Task 0)
├── README.md                     # como usar em 10 linhas (Task 0)
├── VERSION                       # 1.0.0 (Task 6)
├── brand-voice.md                # A voz da INEMA — o arquivo que faz tudo (Task 1)
├── docs/                         # fontes originais + este plano
├── .claude/skills/
│   ├── ms-post/SKILL.md          # Prompt 2 — post do dia (Task 2)
│   ├── ms-semana/SKILL.md        # Prompt 3 — semana no autopilot (Task 4)
│   └── ms-reaproveita/SKILL.md   # Bônus — 1 fonte → N posts (Task 5)
├── pipeline/
│   ├── visual.md                 # receita de geração + formatos + upload (Task 3)
│   └── agendar.md                # receita Metricool: payloads validados (Task 3)
├── posts/                        # 1 pasta por post: YYYY-MM-DD-slug/ {post.md, *.png, status}
│   └── semanas/                  # 1 arquivo por semana: YYYY-WW.md (plano + retro) (Task 4)
└── output/ → ~/projetos/output/ms-v7/   # imagens geradas (regra global de saída)
```

---

### Task 0: Bootstrap do repo e contexto do projeto

**Arquivos:** criar `CLAUDE.md`, `README.md`, `.gitignore`; `git init`.

- [ ] **Passo 1:** `git init` em `~/projetos/ms-v7`; `git config user.name inematds`; `git config user.email inematds@gmail.com`.
- [ ] **Passo 2:** Escrever `CLAUDE.md` com: propósito (1 parágrafo), a tabela "Restrições globais" acima copiada verbatim, os 3 comandos (`/ms-post`, `/ms-semana`, `/ms-reaproveita`), a regra "leia `brand-voice.md` antes de escrever qualquer post", e a regra "nunca chame `createScheduledPost` sem 'sim' do usuário na mesma conversa".
- [ ] **Passo 3:** `.gitignore` com `output/`, `*.mp4`, `*.png` fora de `posts/` (imagens finais aprovadas ficam em `posts/`, rascunhos não).
- [ ] **Passo 4:** `README.md` — o fluxo em 10 linhas: preencher voz uma vez → `/ms-post` → aprovar → agendado.
- [ ] **Passo 5:** `gh auth status`; se a conta ativa não for `inematds`, `gh auth switch -u inematds`.
- [ ] **Passo 6:** Verificar: `git status` limpo após `git add -A && git commit -m "chore: bootstrap ms-v7"`. Criar remoto `gh repo create inematds/ms-v7 --private --source=. --push`.

**Verificação:** `git log --oneline` mostra 1 commit com autor `inematds`; `gh repo view inematds/ms-v7` responde.

---

### Task 1: `brand-voice.md` da INEMA (Prompt 1, uma vez)

**Arquivos:** criar `brand-voice.md` a partir de `docs/brand-voice-template.md`.

**Produz:** o arquivo que TODAS as skills leem. Seções obrigatórias (mesmos títulos do template, em PT-BR): QUEM SOU · PÚBLICO · OFERTAS & CTAs · REGRAS DE VOZ · MINHAS FRASES · MINHAS HISTÓRIAS · PROVA QUE POSSO AFIRMAR · PILARES · ESTILOS POR PLATAFORMA (Instagram, TikTok; LinkedIn/X/Facebook/YouTube mantidos mas marcados `[inativo — não conectado]`).

- [ ] **Passo 1: Pré-preencher** o que já existe, sem perguntar. Fonte: `~/.claude/skills/roteirista-inema/SKILL.md` (bloco "Contexto fixo do canal") e `~/.claude/skills/reel-edita-inema/SKILL.md`. Entram: QUEM SOU (Nei, 40 anos em tecnologia, ensina fundamentos de IA — Claude Code, terminal, documentação — para profissionais 40+ no Brasil), PÚBLICO (profissionais adultos que temem perder relevância pela IA; querem fundamento, não ferramenta), tese central ("IA se aprende lendo, não assistindo"), NUNCA (lista de palavras proibidas + guru), OFERTAS (INEMA.CLUB gratuito → CTA "link na bio"; engajamento → "me conta nos comentários"), PILARES (Claude Code na prática · Ler documentação/system prompts · Mitos de IA · Fundamento vence ferramenta · Bastidores do Nei).
- [ ] **Passo 2: Entrevista em texto livre, UMA pergunta por vez**, só para as lacunas que ninguém pode inventar: (a) 5 frases que o Nei realmente fala; (b) 3 histórias VERDADEIRAS com número e detalhe (uma vitória, um erro, uma surpresa); (c) prova que pode afirmar (anos, alunos, cursos publicados, seguidores); (d) 2 CTAs extras além de "link na bio"; (e) estilo visual (dark âmbar INEMA? terminal na tela? rosto?). Rejeitar resposta vaga ("muitos alunos" → "quantos?").
- [ ] **Passo 3:** Gravar `brand-voice.md`. Regras de plataforma da Fase 1: **Instagram** (gancho ≤ 8 palavras na 1ª linha; linhas curtas; ≤ 5 hashtags no fim; formatos: Reel 30–60s, carrossel 6–10 slides com slide 1 = frase grande, foto única 4:5); **TikTok** (roteiro 30–45s falado + texto na tela; gancho nos 2 primeiros segundos; energia de bastidor).
- [ ] **Passo 4: Teste de voz.** Sem outra fonte além do arquivo, escrever 1 legenda de Instagram sobre "ler documentação". Critérios de aprovação: contém ≥ 1 história/número/frase do arquivo; nenhuma palavra proibida; gancho ≤ 8 palavras; o Nei responde em texto "eu escreveria isso" — se "não", voltar ao Passo 2 na seção que falhou.
- [ ] **Passo 5:** `git commit -m "feat: brand-voice.md da INEMA"`.

**Verificação:** `grep -c "40 anos em IA" brand-voice.md` retorna 0; `grep -c "40 anos em tecnologia"` ≥ 1; teste de voz aprovado.

---

### Task 2: Skill `/ms-post` (Prompt 2 — o post do dia)

**Arquivos:** criar `.claude/skills/ms-post/SKILL.md`.

**Consome:** `brand-voice.md`. **Produz:** pasta `posts/YYYY-MM-DD-<slug>/post.md` com as 3 opções, a escolhida, o visual e o status (`rascunho | aprovado | agendado`).

- [ ] **Passo 1:** Escrever a skill com este fluxo fixo:
  1. Ler `brand-voice.md` inteiro.
  2. Três perguntas, **em texto livre, uma por vez, esperando resposta**: plataforma (Instagram ou TikTok — se disser LinkedIn/X, avisar que não está conectado e oferecer texto copy-paste); tema (algo que aconteceu / algo pra ensinar / oferta / "pesquise o que está em alta no nicho e me traga 5 ângulos"); objetivo (crescer seguidores / DMs e leads / vender / conversa).
  3. Pesquisa (só se pedido ou tema = "em alta"): usar skill `agent-reach` ou `WebSearch` para "o que funciona esta semana" na plataforma (ganchos, formatos, duração). Resumir em 5 linhas, com fontes.
  4. Escrever **3 opções**, cada uma com: ≥ 1 história/número/frase do arquivo; regras da plataforma; ZERO AI-speak; exatamente **1 CTA** compatível com o objetivo. Para Reel/TikTok, a opção é um roteiro no formato do `roteirista-inema` (0-3s gancho / 3-20s / 20-40s / fechamento).
  5. Esperar a escolha (texto). Gravar `posts/<data>-<slug>/post.md`.
  6. Visual: seguir `pipeline/visual.md` (Task 3). Mostrar o caminho do arquivo.
  7. Agendamento: seguir `pipeline/agendar.md`. Mostrar **post final + horário proposto** (via `getBestTimeToPostByNetwork`) e perguntar em texto: "Agendo? (sim/não)". Respostas aceitas: **"sim"** → `createScheduledPost` com `autoPublish: true`; **"rascunho"** → `createScheduledPost` com `draft: true` (fica no planner, não publica); **"não"** ou sem MCP → entregar texto pronto pra copiar.
  8. Atualizar `status` no `post.md` e commitar.
- [ ] **Passo 2:** Incluir na skill a checklist de qualidade que o modelo roda antes de mostrar as opções: gancho ≤ 8 palavras · ≤ 5 hashtags · 1 CTA · sem palavra proibida · sem "40 anos em IA" · sem número inventado · "um seguidor acreditaria que o Nei escreveu?".
- [ ] **Passo 3: Teste seco** (sem agendar): rodar `/ms-post` com plataforma=Instagram, tema="por que ler o system prompt", objetivo=seguidores. Responder "não" no agendamento. Verificar que `posts/` tem a pasta, as 3 opções passam na checklist do Passo 2, e nenhuma chamada a `createScheduledPost` aconteceu.
- [ ] **Passo 4:** `git commit -m "feat: skill /ms-post"`.

---

### Task 3: Pipeline visual + hospedagem + agendamento (receitas)

**Arquivos:** criar `pipeline/visual.md`, `pipeline/agendar.md`.

- [ ] **Passo 1: `pipeline/visual.md`** — receita determinística:
  - Default **flux2-klein**. Prompt base = "estilo visual" do `brand-voice.md` + formato. Tamanhos: Reel/TikTok capa `1080x1920`; feed/carrossel `1080x1350`. Saída em `~/projetos/output/ms-v7/<data>-<slug>/`.
  - Carrossel: slide 1 = frase-gancho grande (texto vem do post, não do gerador de imagem — texto renderizado via HTML→PNG ou sobreposto com Pillow/ImageMagick, porque flux erra tipografia); slides 2..N = 1 ideia por slide; último = CTA.
  - Fallback **Magnific** (ler `~/.claude/runbooks/magnific-modelos.md`): `z-image` 5cr / `flux-2-klein` 10cr para imagem; `kling-25` 140cr para vídeo 5s. Nunca `auto`. `simulate_cost` antes de lote > 10.
  - Vídeo curto sem IA generativa: `pixflow-motion` (foto → movimento) ou `video-explicativo` (HTML→MP4). Música/SFX: inemavox/dlp. Voz: inemavox TTS `chatterbox`/`rachel`.
  - Aprovação visual em texto antes de subir.
- [ ] **Passo 2: `pipeline/agendar.md`** — copiar de `~/projetos/pubmetricool/docs/publicar-video.md` o que já foi validado e fixar:
  ```bash
  # host público (Metricool só aceita URL direta)
  gh release upload v1 <arquivo> --repo inematds/midia --clobber
  # → https://github.com/inematds/midia/releases/download/v1/<arquivo>
  ```
  Ordem obrigatória de chamadas MCP: `getBrandSettings` (confirmar blogId 6745962 + fuso) → `getBestTimeToPostByNetwork(network)` (propor 1 horário) → mostrar post+hora → **"sim"** → `createScheduledPost`. Payloads de referência:
  ```jsonc
  // Reel IG + TikTok
  {"text":"...", "media":["https://github.com/inematds/midia/releases/download/v1/x.mp4"],
   "providers":[{"network":"instagram"},{"network":"tiktok"}],
   "publicationDate":{"dateTime":"2026-09-15T12:00:00","timezone":"America/Sao_Paulo"},
   "autoPublish":true, "draft":false,
   "instagramData":{"type":"REEL","showReelOnFeed":true},
   "tiktokData":{"privacyOption":"PUBLIC_TO_EVERYONE","title":"..."}}
  // Carrossel IG: media = [url1..urlN] (4:5), instagramData {"type":"POST"}
  // ⚠ não validado — pubmetricool só validou REEL + TikTok. Confirmar no primeiro carrossel real.
  ```
  Regras: nunca incluir `{"network":"youtube"}`; registrar `id` + `uuid` retornados no `post.md` (necessários para `updateScheduledPost`); cancelamento é no app da Metricool (não há delete no MCP); contar posts do mês em `posts/` e avisar ao chegar em 18/20.
- [ ] **Passo 3: Teste real, 1 vez** (possivelmente gasta 1 da cota — com ciência do usuário): gerar 1 imagem 4:5 com flux2-klein, subir na release, rodar `/ms-post` e responder **"rascunho"** no gate (→ `draft: true`, fica no planner, não publica). Verificar com `getScheduledPosts` que apareceu (⚠ não validado se essa ferramenta lista rascunhos; se não listar, conferir no planner do app); anotar `id`/`uuid`. Conferir depois no app se o contador mensal subiu.
- [ ] **Passo 4:** `git commit -m "feat: pipeline visual + agendar"`.

**Verificação:** `getScheduledPosts` lista o rascunho; a URL da release responde `200` em `curl -sI`.

---

### Task 4: Skill `/ms-semana` (Prompt 3 — a semana no autopilot)

**Arquivos:** criar `.claude/skills/ms-semana/SKILL.md`; criar `posts/semanas/YYYY-WW.md`.

**Consome:** `brand-voice.md`, `pipeline/*.md`, analytics do Metricool. **Produz:** plano semanal em tabela, fila de posts, retro da semana anterior.

- [ ] **Passo 1:** Fluxo da skill:
  1. **Retro** (a partir da 2ª semana): `getAnalyticsAvailableMetrics(instagram, posts)` e `(instagram, reels)`, `(tiktok, posts)` → IDs → `getAnalyticsDataByMetrics` dos últimos 7 dias (**um conector por chamada**, janela ≤ 30 dias). Dizer em 5 linhas quais posts foram melhor e o que muda esta semana.
  2. Três perguntas em texto livre, uma por vez: plataformas da semana; temas/notícias da semana do Nei (ou "pesquise ângulos"); objetivo nº 1.
  3. Planejar **5–7 posts**, respeitando a cota mensal restante (contar em `posts/`). Mix Instagram: 2 Reels + 2 carrosséis + 1 foto; TikTok: 2 roteiros. Mostrar **tabela** (dia · hora sugerida por `getBestTimeToPostByNetwork` · plataforma · formato · gancho · CTA) e **esperar edições**.
  4. Após "ok": escrever todos os posts (mesmas regras do `/ms-post`), gerar visuais (`pipeline/visual.md`), mostrar a **fila completa** (dia, hora, plataforma, texto, visual).
  5. Agendar **só após "sim" explícito**, um `createScheduledPost` por post, registrando `id`/`uuid` em cada `posts/<data>-<slug>/post.md`.
- [ ] **Passo 2: Teste seco:** rodar até a tabela do item 3 e parar. Verificar: ≤ 7 linhas, soma ≤ cota restante, nenhuma linha YouTube, cada linha tem 1 CTA.
- [ ] **Passo 3:** `git commit -m "feat: skill /ms-semana"`.

---

### Task 5: Skill `/ms-reaproveita` (Bônus — uma fonte vira tudo)

**Arquivos:** criar `.claude/skills/ms-reaproveita/SKILL.md`.

- [ ] **Passo 1:** Fluxo: entrada = caminho de transcrição/artigo/notas ou URL (se URL, buscar com `agent-reach`; se YouTube próprio, transcrição via skill `watch`). **Única pergunta** em texto: quais plataformas. Extrair os 5 momentos mais surpreendentes/específicos. Gerar 1 post por plataforma, cada um **autônomo** (nunca "neste vídeo"), regras do `brand-voice.md`. Depois seguir a fila igual ao `/ms-semana` (mostrar tudo, nada vai sem "sim").
- [ ] **Passo 2: Teste seco:** entrada = `docs/social-media-automation-fable-5.md` (a própria transcrição), plataformas = Instagram. Verificar: nenhum "neste vídeo"; cada post cita um momento específico; 1 CTA.
- [ ] **Passo 3:** `git commit -m "feat: skill /ms-reaproveita"`.

---

### Task 6: Fechamento da Fase 1

- [ ] **Passo 1:** `VERSION` = `1.0.0`; registrar em `CLAUDE.md`.
- [ ] **Passo 2:** Rodar `/ms-post` de ponta a ponta com um post REAL aprovado pelo Nei (Instagram, `autoPublish: true`). Confirmar no perfil que publicou na hora marcada.
- [ ] **Passo 3:** Criar `FALHAS.md` (formato `| data | o que quebrou | menor correção | prompt \| infra |`) e registrar qualquer falha ocorrida nos testes.
- [ ] **Passo 4:** `git commit -m "release: v1.0.0" && git push`.

**Verificação de conclusão:** push em `origin`; 1 post publicado de verdade; `posts/` com o histórico e `id/uuid`.

---

## Fase 2 (não planejada em detalhe — só registrar)

- **Mais redes:** Starter da Metricool (€16/mês) libera LinkedIn; X é add-on. Ou Blotato ($29, API inclusa) se DMs/comentários virarem requisito — ver `pubmetricool/docs/comparativo-blotato.md`.
- **Vídeo gerado por IA:** `kling-25` via Magnific (140cr/5s) quando um Reel pedir cena impossível de filmar.
- **Agendamento automático (cron):** skill `schedule` do Claude Code rodando `/ms-semana` toda segunda 8h — só depois de o fluxo manual rodar 4 semanas sem falha.
- **Relação com `timesmkt2` (ITAGMKT v4.3.x):** assumido que ms-v7 é um reinício leve, deliberado, sem herdar o pipeline de 13 agentes. Se for pra reaproveitar algo de lá, os candidatos são os specs de plataforma em `timesmkt2/skills/`.

## Perguntas em aberto (responder em texto antes da Task 1)

1. **Qual voz e qual marca?** A marca da Metricool é `inemafuturos` (IG) + `@lnema.club` (TikTok). A voz que vou semear é a do **INEMA.TDS / Nei** (roteirista-inema). É a mesma persona nessas contas, ou o `@inemafuturos` tem outra voz e precisa de um segundo `brand-voice.md`?
2. **YouTube:** deixo fora do agendamento (recomendado), ou reaponto a marca para o INEMA TIA (`UCavuQHkxBSAZbzRoOm6Gq4g`)?
3. **Teste real da Task 3** gasta 1 dos 20 posts do mês (fica como rascunho, não publica). Ok?
4. **Rosto no visual:** as imagens podem usar o avatar/foto do Nei (referência de personagem via Magnific) ou ficam só tipografia/terminal no estilo dark âmbar INEMA?
