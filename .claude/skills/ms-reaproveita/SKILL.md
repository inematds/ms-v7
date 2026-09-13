---
name: ms-reaproveita
description: Uma fonte vira tudo (bônus do Social Autopilot). Recebe transcrição de vídeo, artigo, notas de podcast ou URL, pergunta só quais redes, e gera um post nativo e autônomo por rede na voz da marca, depois entra na fila de aprovação. Use para "reaproveita esse vídeo", "transforma esse artigo em posts", "faz posts a partir dessa transcrição".
---

# /ms-reaproveita — Uma coisa vira tudo

Mesmo carregamento de contexto do `/ms-post` (passo 0).

## 1. Entrada

- Texto colado, caminho de arquivo, ou URL.
- URL de artigo/post → buscar com o provedor de pesquisa (`agent-reach`).
- URL de vídeo do YouTube → transcrição via skill `watch`.
- Arquivo de vídeo/áudio local → transcrever (inemavox ou whisper) antes.

## 2. A única pergunta

"Para quais redes?" — listar só as ativas. Se já veio na mensagem, não perguntar.

## 3. Extrair

Ler a fonte inteira e listar os **5 momentos mais surpreendentes ou específicos** (número, história, frase, opinião contrária). Mostrar a lista em 5 linhas. Não perguntar nada — seguir.

## 4. Gerar um post por rede

- Cada peça é **autônoma**: nunca "neste vídeo", "como falei no artigo", "no episódio de hoje".
- Segue a seção da rede em `brand-voice.md` + regras de voz + 1 CTA.
- Usa pelo menos 1 dos 5 momentos por post; redes diferentes usam momentos diferentes quando possível.
- Checklist do `/ms-post` passo 3.

Gravar cada um em `posts/<data>-<slug>-<rede>/post.md` com `fonte: <caminho ou URL>` no frontmatter.

## 5. Fila → aprovação → publicar

Igual ao `/ms-semana` passos 4-6: visuais (se configurado), fila completa, `Agendo? (sim / rascunho / não)`, nada sem "sim", commit ao final.
