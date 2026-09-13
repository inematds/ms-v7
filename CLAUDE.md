# ms-v7 — Social Autopilot (configurável)

**Versão:** 1.0.0

Sistema de publicação em redes sociais dirigido por um arquivo de voz de marca + 3 prompts, adaptado de "The Social Autopilot" (Zubair Trabzada, ver `docs/`). Sem código de aplicação: tudo é Claude Code + skills locais + provedores plugáveis via `config.yaml`.

## Mapa

| O quê | Onde |
|---|---|
| Marca ativa, redes, provedores, regras | `config.yaml` |
| Voz de cada marca | `brands/<slug>/brand-voice.md` (template em `brands/_template/`) |
| Como publicar em cada provedor (Metricool validado, Blotato, copiar) + hospedagem | `pipeline/publicar.md` |
| Como gerar imagem/vídeo (flux2-klein, Magnific, pixflow…) | `pipeline/visual.md` |
| Histórico de posts | `posts/<AAAA-MM-DD>-<slug>/post.md`; semanas em `posts/semanas/` |
| Fontes originais e plano | `docs/` |

## Comandos

| Skill | Quando |
|---|---|
| `/ms-config` | Primeira vez, nova marca, ativar rede, trocar provedor |
| `/ms-post` | O post de hoje (3 perguntas → 3 opções → visual → agendar) |
| `/ms-semana` | Retro + plano de 5-7 posts + fila da semana |
| `/ms-reaproveita` | Transcrição/artigo/URL → um post por rede |

## Regras fixas (não mudam por marca)

1. **Ler `config.yaml` e o `brand-voice.md` da marca ativa antes de escrever qualquer post.** Se `marca_ativa: null`, mandar rodar `/ms-config`.
2. **Nada é agendado ou publicado sem "sim" (ou "rascunho") explícito do usuário nesta conversa.** Nunca chamar `createScheduledPost` ou equivalente de outra forma.
3. Só redes com `ativa: true` recebem posts. Rede inativa → texto em modo `copiar`, com aviso.
4. Perguntas ao usuário sempre em texto livre, uma por vez. Nunca menu interativo.
5. Nunca inventar número, história ou estatística. Só o que está no `brand-voice.md` ou o usuário disse.
6. Respeitar `cota_mensal`; avisar em 90%.
7. Imagem: provedor padrão do `config.yaml` (flux2-klein). Magnific só como fallback e nunca em `auto`.
8. Cada post/semana termina em commit. Publicar o projeto = push em `origin` (repo `inematds/ms-v7`, autor `inematds <inematds@gmail.com>`).
9. Falha real → uma linha em `FALHAS.md` antes da próxima tarefa.

## Estado dos provedores

| Provedor | Status |
|---|---|
| Metricool MCP | validado 2026-08-19 (Reel IG + TikTok); carrossel não validado |
| Blotato | não testado |
| flux2-klein | padrão global |
| Magnific | fallback, com crédito |
| GitHub Release `inematds/midia` | validado como host |
