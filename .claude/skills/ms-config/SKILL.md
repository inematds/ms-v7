---
name: ms-config
description: Configura o Social Autopilot — cria/edita uma marca em brands/<slug>/ (entrevista de voz de marca, uma pergunta por vez) e preenche config.yaml (redes ativas, provedor de publicação, imagem, vídeo). Use para "configura minha marca", "me entrevista pra preencher minha voz", "troca o provedor", "ativa o LinkedIn", "cria outra marca".
---

# /ms-config — Configurar marca, redes e provedores

Perguntas SEMPRE em texto livre, **uma por vez**, esperando a resposta. Nunca usar menu interativo.

## Modo A — Nova marca (ou `marca_ativa: null`)

1. Perguntar o **slug** da marca (ex.: `inema`). Criar `brands/<slug>/` copiando `brands/_template/brand-voice.md`.
2. **Pré-preencher o que já existe** antes de perguntar: se o usuário citar um projeto/skill/site que já descreve a marca, ler de lá (ex.: para INEMA, `~/.claude/skills/roteirista-inema/SKILL.md` já tem público, tese, palavras proibidas, produtos). Mostrar o que foi pré-preenchido e pedir correção.
3. **Entrevista** só das lacunas, nesta ordem, uma pergunta por vez, **rejeitando resposta vaga** ("muitos alunos" → "quantos?"; "sou bom nisso" → "que número prova?"):
   - QUEM SOU em uma frase
   - PÚBLICO: quem é, o que quer e não tem
   - OFERTAS & CTAs: 2 a 4, com a frase exata do CTA
   - VOZ: os 4 eixos + o que NUNCA postar + palavras proibidas
   - 5 FRASES que a pessoa realmente fala
   - 3 HISTÓRIAS verdadeiras (vitória, erro, surpresa) com número e detalhe
   - PROVA verificável
   - 3-5 PILARES
   - ESTILO VISUAL (paleta, o que aparece, o que nunca aparece)
4. Gravar `brands/<slug>/brand-voice.md`. Manter as seções de plataforma do template; marcar `[inativa]` nas redes não ativas.
5. **Teste de voz:** sem outra fonte, escrever 1 legenda de Instagram sobre o pilar 1. Perguntar: "Você escreveria isso?" Se não, voltar à seção que falhou.

## Modo B — Redes e provedores (`config.yaml`)

Uma pergunta por vez:
1. Quais redes têm conta? Para cada: handle e se está **ativa** para receber posts. Gravar em `marcas.<slug>.redes`.
2. Provedor de publicação: `metricool` (já validado aqui, Free), `blotato` (não testado) ou `copiar` (sem API). Se MCP: pedir para o usuário conectar (`claude mcp add ...` em `pipeline/publicar.md`) e confirmar com "list my connected MCP tools" / `getBrandSettings`. Gravar `blogId`, plano e `cota_mensal`.
3. Imagem: manter `flux2-klein` (padrão global) salvo pedido contrário. Vídeo: `nenhum` até a marca precisar.
4. Fuso e idioma.
5. Gravar `config.yaml`, definir `marca_ativa: <slug>`, mostrar o YAML final.

## Modo C — Ajuste pontual

"Ativa o LinkedIn", "troca pra blotato", "usa a marca X": editar só a chave pedida em `config.yaml`, mostrar o diff, e avisar se o provedor exige plano/conexão (ex.: LinkedIn não existe no Metricool Free).

## Sempre ao terminar

- Validar: `python3 -c "import yaml,sys; yaml.safe_load(open('config.yaml'))"`.
- `git add -A && git commit -m "config: marca <slug>"`.
- Dizer em 3 linhas: marca ativa, redes ativas, provedor de publicação, e o próximo comando (`/ms-post`).
