# skills

A skill database. One canonical copy of every agent skill worth keeping,
pulled in from Claude Code, Codex, OpenCode, Gemini/Antigravity and Cursor so
they live in one place instead of five config directories.

**167 skills** across 25 entries.

## Bundles

Multi-skill plugins kept whole — their skills reference sibling `scripts/`,
`hooks/` and `agents/` by relative path, so splitting them breaks the refs.

| Bundle | Skills | What's in it |
|---|---:|---|
| [marketing-skills](./marketing-skills) | 49 | Full marketing stack — seo-audit, ai-seo, programmatic-seo, copywriting, cro, ads, emails, pricing, launch, analytics, attribution, churn-prevention, marketing-psychology, and more |
| [mattpocock-skills](./mattpocock-skills) | 35 | Matt Pocock's set, organised into `engineering/`, `productivity/`, `misc/`, `in-progress/` — tdd, diagnosing-bugs, codebase-design, domain-modeling, code-review, research, wizard, grilling, writing-for-agents |
| [claude-seo](./claude-seo) | 33 | SEO suite — technical, schema, geo, local, maps, cluster, content-brief, dataforseo — plus 18 subagents, hooks and scripts |
| [superpowers](./superpowers) | 14 | brainstorming, executing-plans, systematic-debugging, TDD, writing-plans, writing-skills, using-git-worktrees, subagent-driven-development, verification-before-completion |
| [ponytail](./ponytail) | 12 | Laziest-solution-that-works enforcement — ponytail, -audit, -debt, -gain, -help, -review, plus commands, hooks and a statusline |
| [chrome-devtools](./chrome-devtools) | 5 | Chrome DevTools MCP suite — chrome-devtools, debug-optimize-lcp, a11y-debugging, memory-leak-debugging, troubleshooting |

## Standalone skills

| Skill | Description |
|---|---|
| [gws-drive](./gws-drive) | Google Drive v3 via the official `gws` CLI — files, permissions, folders, comments, revisions, shared drives, changes. Includes `references/` covering install (Arch AUR / Debian APT / npm), OAuth setup, and every error hit during setup |
| [emil-design-eng](./emil-design-eng) | Emil Kowalski's philosophy on UI polish, component design and animation |
| [frontend-design](./frontend-design) | Distinctive, intentional visual design for new or reshaped UI |
| [scrollcraft](./scrollcraft) | Scroll-driven interaction and animation design |
| [skill-creator](./skill-creator) | Authoring new skills — the meta-tool for this repo |
| [session-handoff](./session-handoff) | Structured end-of-session summary before clearing context |
| [cloudflare](./cloudflare) | Umbrella Cloudflare platform skill — Workers, Pages, KV, D1, R2, AI, security |
| [agents-sdk](./agents-sdk) | Stateful agents on Workers — Agent class, Workflows, queues, React hooks |
| [durable-objects](./durable-objects) | Durable Objects — RPC, SQLite storage, alarms, WebSockets |
| [workers-best-practices](./workers-best-practices) | Production Workers review — streaming, floating promises, bindings, secrets |
| [wrangler](./wrangler) | Workers CLI — deploy, dev, KV/R2/D1/Vectorize/Queues/Secrets |
| [cloudflare-email-service](./cloudflare-email-service) | Transactional email — Email Sending + Routing, SPF/DKIM/DMARC |
| [cloudflare-one](./cloudflare-one) | Zero Trust / SASE — Access, Gateway, WARP, Tunnel, DLP, CASB |
| [cloudflare-one-migrations](./cloudflare-one-migrations) | Migrating from Zscaler, Palo Alto, legacy VPN/SWG to Cloudflare One |
| [sandbox-stable](./sandbox-stable) | Cloudflare Sandbox on the stable `@cloudflare/sandbox` package |
| [sandbox-next](./sandbox-next) | Cloudflare Sandbox on `@next` (SDK 1.0 preview) |
| [sandbox-migrate-to-next](./sandbox-migrate-to-next) | Porting a Sandbox app from stable to `@next` |
| [turnstile-spin](./turnstile-spin) | End-to-end Turnstile setup — widget creation, embed, server-side siteverify |
| [web-perf](./web-perf) | Core Web Vitals audit via Chrome DevTools MCP — LCP, INP, CLS |

## Browse

`tree` is unusable here — 1972 entries. Two views instead:

```sh
# shape (27 lines)
tree -L 1

# skills per entry (25 lines)
for d in */; do printf '%3d  %s\n' "$(find "$d" -name SKILL.md | wc -l)" "${d%/}"; done | sort -rn

# every skill path (167 lines)
find . -name SKILL.md | sed 's|/SKILL.md$||; s|^\./||' | sort
```

`tree --filelimit N` does not help: it clamps the root too, so 27 top-level
entries collapse the whole listing to one line.

## Layout

Standalone skills sit at the top level, one directory per skill, each with a
`SKILL.md` carrying `name` and `description` frontmatter. Bundles keep their
original plugin structure (`skills/`, `agents/`, `hooks/`, `scripts/`).

## Install

Copy the directory you want into your agent's skills path:

```sh
cp -r <skill> ~/.claude/skills/        # Claude Code
cp -r <skill> ~/.codex/skills/         # Codex
cp -r <skill> ~/.opencode/skills/      # OpenCode
```

For bundles, copy the inner `skills/*` directories, or install the whole
bundle as a plugin.

## License

Each skill retains the license of its upstream source (see per-directory
`LICENSE` where present). Repo scaffolding is MIT.
