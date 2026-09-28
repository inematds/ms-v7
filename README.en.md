# ms-v7 — Social Autopilot

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

A brand voice file + 3 prompts → native posts for each network, in your voice, scheduled through the provider you choose. **Nothing goes live without your “yes.”**

Adapted from *The Social Autopilot* (Zubair Trabzada, AI Workshop). The originals (prompt pack, voice template, video transcript) are in `docs/`. The implementation plan is in `docs/PLANO.md`.

## 📖 User guide

Full guide (landing page + walkthrough): **https://inematds.github.io/ms-v7/guia/en/**

---

## 1. How it works (in 1 minute)

```
brand-voice.md  ─┐
config.yaml     ─┼─►  /ms-post  ──► 3 questions ──► 3 options ──► visual ──► "Schedule? (yes/draft/no)"
providers       ─┘        │
                          ├─► /ms-semana       weekly recap + table of 5-7 posts + week’s queue
                          └─► /ms-reaproveita  transcript / article / URL → one post per network
```

- **Brand voice** (`brands/<slug>/brand-voice.md`): who you are, your audience, offers, phrases, true stories, and rules for each network. It makes the post sound like you, not AI.
- **Config** (`config.yaml`): which brand is active, which networks receive posts, who publishes (Metricool, Blotato, or manual copy), and who generates images/videos.
- **Skills** (`.claude/skills/`): the prompt pack packaged as Claude Code commands.
- **Pipelines** (`pipeline/`): step-by-step recipes for each provider.

---

## 2. Start (prerequisites)

| What | How | Required? |
|---|---|---|
| Claude Code | `claude.com/claude-code` — Pro plan or higher | yes |
| This repo | `git clone git@github.com:inematds/ms-v7.git ~/projetos/ms-v7` | yes |
| Python 3 + PyYAML | `pip install pyyaml` (only to validate `config.yaml`) | recommended |
| Publishing provider | Metricool (MCP, free) or Blotato (MCP, paid) — see §4 | no (without one, `copy` mode) |
| Image generator | local flux2-klein (default) or Magnific (MCP) | no |
| Media host | `gh` authenticated with the `inematds` account (release in `inematds/midia`) | only with MCP provider |

Open Claude Code **inside the folder**:

```bash
cd ~/projetos/ms-v7
claude
```

It reads `CLAUDE.md` automatically and learns the rules and commands.

---

## 3. Configure your first brand

In Claude Code:

```
/ms-config
```

The command asks **one thing at a time**, in free text. It has three sections:

**A. Brand voice** → creates `brands/<slug>/brand-voice.md` from `brands/_template/`.
- If the brand is already described somewhere (a skill, website, or doc), say where: it pre-fills the file and asks only for what’s missing.
- What it always asks because no one can make it up: 5 phrases you actually say, 3 true stories with numbers, verifiable proof, exact CTAs.
- Vague answers are rejected (“many students” → “how many?”).
- It ends with a **voice test**: it writes a caption using only the file and asks, “Would you write this?”

**B. Networks and providers** → fills in `config.yaml`.
- For each network: handle and whether it’s `active`. Only active networks receive posts.
- Publishing provider: `metricool` · `blotato` · `copy`.
- Image: `flux2-klein` (default) or `magnific`. Video: `none` until needed.

**C. Adjustments later**: “activate LinkedIn,” “switch to blotato,” “use brand X” — it edits only the requested key.

You can also edit `config.yaml` by hand. Validate it:

```bash
python3 -c "import yaml; yaml.safe_load(open('config.yaml'))" && echo ok
```

### Multiple brands

Each brand is a folder in `brands/`. Only one is active (`marca_ativa` in `config.yaml`). To switch: “use brand X” or edit the key.

---

## 4. Connect a publishing provider

### Metricool (validated, Free plan)

```bash
claude mcp add --transport http metricool https://ai.metricool.com/mcp -s user
```

Inside Claude Code: `/mcp` → **metricool** → **Authenticate** → authorize in the browser. Then, in `/ms-config`, enter the `blogId` (it finds it with `getBrandSettings`).

Free plan limits: 1 brand, 1 account per network, **no LinkedIn or X**, **20 posts/month**, 30-day analytics, no “delete post” via MCP (cancel in the app).

### Blotato (not tested here)

Create an account → connect networks → copy the MCP URL → `claude mcp add --transport http blotato <url> -s user`. Before using it in production, create a post as a `draft` and note the working payload in `pipeline/publicar.md`.

### Copy (no API)

Nothing to connect. The skills provide the ready-to-use text in a code block, and you paste it into the network. This is the default while no provider is configured.

### Media host

MCP providers accept **direct image/video URLs** only. The default is a GitHub Release:

```bash
gh release upload v1 arquivo.png --repo inematds/midia --clobber
# → https://github.com/inematds/midia/releases/download/v1/arquivo.png
```

The skills do this automatically; you only need `gh` logged into the `inematds` account.

---

## 5. Daily use

| Command | What happens | Time |
|---|---|---|
| `/ms-post` | 3 questions (network, topic, goal) → optional research → 3 options → you choose → visual → “Schedule?” | 3-5 min |
| `/ms-semana` | Last week’s recap (analytics) → 3 questions → table of 5-7 posts → you edit/approve → writes everything → visuals → “Schedule the queue?” | 15 min |
| `/ms-reaproveita` | Paste transcript/article/URL → “which networks?” → one standalone post per network → queue | 5 min |

**The approval gate** accepts three responses:

- `yes` → schedules with automatic publication at the proposed time.
- `draft` → creates a draft with the provider (does not publish).
- `no` → text only, for copying.

Each post is saved in `posts/<data>-<slug>/post.md` with its status, provider `id`/`uuid`, and approved visual. Weeks are saved in `posts/semanas/`.

---

## 6. Publish to git

The work **ends with the push**. Deployment is the webhook’s responsibility, not yours.

```bash
cd ~/projetos/ms-v7
git config user.email   # must be inematds@gmail.com
git add -A
git commit -m "post: 2026-09-15-ler-documentacao (instagram, agendado)"
git push
```

Rules:
- Repo: `inematds/ms-v7`. Author and committer: `inematds <inematds@gmail.com>`. If `git config user.email` differs, fix it **locally** (`git config user.email inematds@gmail.com`), without changing the global config.
- The skills already commit at the end of each post/week. You only run `git push`.
- Videos and images outside `posts/` are ignored by `.gitignore`. Public media goes in a release in the `inematds/midia` repo, never in this repo’s history.
- Version in `VERSION` and at the top of `CLAUDE.md` (`vX.XX.YY`: patch increments `YY`; feature increments `XX` and carries over `YY`; only a major resets).
- Actual failure → one line in `FALHAS.md` (`| date | what broke | smallest fix | prompt \| infra |`).

---

## 7. Publish to the portal (inema.club)

The portal points to a **landing page + guide** served by GitHub Pages **from this same repo** (never a separate repo).

**Step 1 — create the guide** (it already exists at `guia/index.html`; regenerate if the project changes):

```
/projetos-landing-guia
```

Generates `guia/index.html` (self-contained, INEMA dark amber style) with `guia/assets/` alongside it.

**Step 2 — enable GitHub Pages via GitHub Actions** (the `.github/workflows/pages.yml` workflow is already in the repo; don’t use the “legacy” branch build, which gets stuck):

```bash
gh repo edit inematds/ms-v7 --visibility public --accept-visibility-change-consequences   # Pages requires a public repo on the free plan
gh api -X POST repos/inematds/ms-v7/pages -f build_type=workflow \
  || gh api -X PUT repos/inematds/ms-v7/pages -f build_type=workflow
git add guia && git commit -m "docs: guia landing" && git push   # the push triggers deployment
gh run list --repo inematds/ms-v7 --limit 3                       # monitor
```

> Commits that touch `.github/workflows/` must be pushed over SSH (`git@github.com:inematds/ms-v7.git`): the `inematds` account’s HTTPS token doesn’t have the `workflow` scope. This repo’s remote is already SSH.

Resulting URL: `https://inematds.github.io/ms-v7/guia/en/`. Check with `curl -sI <url> | head -1` (should return 200).

> Before making the repo public: `brands/` contains your brand voice and personal stories, and `posts/` contains the history. If you don’t want that public, move the guide to a `gh-pages` branch containing only `guia/`, or keep the repo private and host the guide somewhere else.

**Step 3 — add it to the portal:**

```
/atualiza-portal https://inematds.github.io/ms-v7/guia/en/
```

The skill creates the card on all 3 INEMA surfaces (portal `inema.club`, inemabuscas, and the PRO catalog), commits, and pushes to all 3 repos. Vercel deploys automatically via the webhook; there’s no need (and you shouldn’t) check the dashboard.

---

## 8. Repo map

```
ms-v7/
├── CLAUDE.md                  fixed rules + commands (Claude Code reads it automatically)
├── README.md                  this file
├── VERSION                    1.0.0
├── FALHAS.md                  failure log, one line each
├── config.yaml                active brand · networks · providers · rules
├── brands/
│   ├── _template/brand-voice.md   voice template (PT-BR, all networks)
│   └── <slug>/brand-voice.md      one folder per brand
├── pipeline/
│   ├── publicar.md            Metricool (validated) · Blotato · copy · hosting
│   └── visual.md              formats per network · flux2-klein · Magnific · video
├── posts/
│   ├── <AAAA-MM-DD>-<slug>/post.md   each post, with status and id/uuid
│   └── semanas/<AAAA-WW>.md          plan + recap for each week
├── .claude/skills/
│   ├── ms-config/   ms-post/   ms-semana/   ms-reaproveita/
├── guia/                      landing page + guide (GitHub Pages) → inematds.github.io/ms-v7/guia/
└── docs/                      original prompt pack + PLANO.md
```

---

## 9. Common issues

| Symptom | Likely cause | Fix |
|---|---|---|
| “no brand configured” | `marca_ativa: null` | `/ms-config` |
| Skill returns text only, doesn’t schedule | provider = `copiar` or MCP disconnected | `/mcp` → Authenticate, or `/ms-config` “switch to metricool” |
| `createScheduledPost` rejects media | URL isn’t a direct file or doesn’t return 200 | `curl -sI <url>`; redo `gh release upload` |
| Duplicate post on YouTube | `youtube` network active for a brand whose channel is the source of another pipeline | `ativa: false` for youtube in `config.yaml` |
| Empty analytics in the recap | Free plan’s 30-day window, or wrong connector | see `pipeline/publicar.md` § Analytics |
| Push rejected due to author | wrong `user.email` | `git config user.email inematds@gmail.com` and empty commit |
