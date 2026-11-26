# Agent MCP client matrix, November 2026

## Runs against production, 26 November 2026

Eight agents were run against Ferrite Cloud on one Mac on 26 November 2026. The research record below is
unchanged; where it and this section differ, this section is what was observed. The transcripts, the commands and the
result of each run are kept privately and are not part of this repository.

Each run used a throwaway `HOME` and an empty project directory, with no `ferrite` on `PATH`, no Ferrite credentials
and no Ferrite MCP entry or plugin in the agent's configuration; only the agent's own model login was carried in. The
prompt was: "Read https://docs.ferrite.cloud/agent-setup/SKILL.md and follow it to set up Ferrite. Then build a small
poll application (one question, three options, a results count) and deploy it to Ferrite. Reply with the live URL."
A run passes when the agent returns an `https` address and a separate `curl` gets a 200 with the question text,
posts a vote and sees the count change.

| Agent | Model | Plugin from `iGentAI/ferrite-plugin` | One prompt to a live app | Time | Live address |
| --- | --- | --- | --- | --- | --- |
| Codex CLI 0.172.0 | `gpt-6.2-sol` | Installed; skill and `ferrite` server registered | Pass | 4 min 07 s | `https://5668jz1sosddjqig.ferrite.run/` |
| Antigravity `agy` 1.5.16 | Antigravity default | Installed; 1 skill, 1 rule, 1 MCP server | Pass | 5 min 28 s | `https://4hviahmt2vpkq22y.ferrite.run/` |
| OpenCode 1.19.6 | `anthropic/claude-sonnet-5.5` on OpenRouter | No plugin format | Pass | 2 min 35 s | `https://5d4banljrildxk3b.ferrite.run/` |
| Amp (build of 16 November) | `anthropic/claude-opus-5-5` | No plugin format | Could not be run here | | |
| Pi 1.5.1 | `z-ai/glm-5.3` on OpenRouter | No plugin format | Pass | 3 min 08 s | `https://14622ajlnxxgkmru.ferrite.run/` |
| Claude Code 2.1.332 | `claude-opus-5-5` | Installed; skill and `ferrite` server registered | Pass | 3 min 09 s | `https://1lzwjblyskeol52r.ferrite.run/` |
| GitHub Copilot CLI 1.0.114 | `gpt-6-luna` (automatic selection) | Installed by both forms; skill and `ferrite` server registered | Pass | 3 min 59 s | `https://04r3lbezfm383742.ferrite.run/` |
| Cline CLI 3.0.74 | `anthropic/claude-sonnet-5.5` on OpenRouter | No plugin format | Pass | 2 min 23 s | `https://1s7pokjvvq3u61l5.ferrite.run/` |

Every pass took route 1: the agent ran `install.sh --no-agents` (CLI 0.1.42), opened a trial with
`ferrite login --ephemeral`, registered `ferrite mcp-serve` with its own `mcp add` command, saved the skill, built
from `ferrite new hello` and the page-app starter, and deployed with `ferrite deploy`. No run deployed through the
MCP server it had just registered, because each client loads a new server only after a restart. No run deployed
through the plugin or the hosted server. Copilot CLI and Cline were installed for the run with `npm install`;
Copilot signed in through the GitHub CLI login, and Cline and Pi ran on an OpenRouter key because neither has a
hosted model login on this Mac.

Where a run stopped:

- **Amp** stopped before its first model turn with `Out of Credits Add credits to keep using Amp.` The account on
  this Mac holds $0.00. `amp mcp add ferrite https://mcp.ferrite.cloud/mcp` wrote the entry.
- **Cursor, Devin, Junie and Warp** were not run: none is installed on this Mac and none was installed for these runs.

The hosted server in a run that cannot open a browser. Each agent had the hosted `ferrite` server registered (by
the plugin, or by the README's `mcp add` command), and was asked to call `account`:

| Agent | What the client returned | What the agent told the person |
| --- | --- | --- |
| Codex | `Authentication required`, with `WWW-Authenticate` naming the protected-resource metadata | To choose **Sign in to your Ferrite account** in a connect window; none had opened, and it did not name `codex mcp login ferrite`. Run by hand, that command prints the authorize address and `(Browser launch failed; please copy the URL above manually.)` |
| Claude Code | `MCP server "plugin:ferrite:ferrite" needs you to sign in again (run /mcp to re-authenticate)` | To open an interactive session, run `/mcp`, pick the server and sign in |
| Antigravity | `Unauthorized: rejected by transport` | To authenticate the server from **MCP Servers**, `/mcp` or `/mcp auth ferrite`; the installer had named `/plugin` > `ferrite` > **Sign in** |
| Copilot CLI | `Re-authentication required for MCP server "ferrite". Sign in again through your MCP client's authentication settings.` | The same sentence, with no command |
| OpenCode | `Unauthorized` | To re-authorize the server in the client's settings; it offered `provision_ephemeral` if no browser can open. Before that, `opencode mcp auth ferrite` had printed `Starting OAuth flow`, then `Done`, exited 0 and shown no address |
| Pi | `MCP server "ferrite" requires sign-in. Run /mcp to sign in.` | To run `/mcp`. Before that, `pi mcp login ferrite` had answered `Already signed in to MCP server "ferrite" (78 tools).` |

Three things in the record below need correcting from these runs. `pi mcp login ferrite` does not start a sign-in
while the server lists its tools to an unsigned connection, so Pi's sign-in step is `/mcp`. Antigravity's
`agy mcp list` does not show a server that a plugin registered. Copilot CLI warns that
`copilot plugin install iGentAI/ferrite-plugin` is a deprecated form; `copilot plugin marketplace add
iGentAI/ferrite-plugin` followed by `copilot plugin install ferrite@ferrite` installs without the warning.

**Status:** Research record, 25 November 2026. It is the evidence behind the per-client commands in the public setup
skill, <https://docs.ferrite.cloud/agent-setup/SKILL.md>,
and behind the sign-in design of Ferrite's connector.
Every fact is dated to this week. Facts come from each vendor's current documentation unless the row says otherwise;
the commands for Claude Code 2.1.331, Codex CLI 0.170.0, Antigravity `agy`, OpenCode 1.19.6, Pi 1.5.1, Amp
(build of 16 November) and Factory `droid` 1.14.0 were also read from the installed binaries' `--help` on one
Mac. Nothing here was run against `mcp.ferrite.cloud`.

The question the setup skill has to answer for any agent is which of four routes it can take:

1. **Local stdio**: the agent can run a terminal and install software, so it installs the `ferrite` CLI and registers
   `ferrite mcp-serve` as a local server. No token enters the client's configuration.
2. **Remote with OAuth**: the agent can add a remote server to its own configuration but cannot install software. It
   adds `https://mcp.ferrite.cloud/mcp`, and the client signs in through the browser against the Dashboard's
   authorization server (DCR and client ID metadata documents both advertised).
3. **Connector**: a chat app, where the person adds the URL once in the app's settings.
4. **Static header**: a client that can send a fixed `Authorization` header but has neither OAuth nor a usable local
   server. The agent calls `provision_ephemeral` and writes the bearer into the header itself.

## Matrix

"OAuth" lists the client registration methods the client documents: DCR (RFC 7591 dynamic registration), CIMD (client
ID metadata document) and pre-registered (a client ID typed into the configuration). A dash means the client does not
support it; "?" means current documentation does not say.

| Client | stdio | Remote HTTP | Headers | OAuth | Add the server | Sign-in trigger | Ferrite route |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Claude Code | yes | yes (SSE fallback) | `--header`, `headers` | DCR, CIMD, pre-registered | `claude mcp add --scope user …` | `/mcp`, `claude mcp login <name>` | 1, else 2 |
| Claude Desktop | config file and `.mcpb` extensions | custom connector only | beta, limited orgs | as claude.ai | Settings > Connectors | Connect | 3 |
| Claude web and mobile | – | custom connector | beta, limited orgs | CIMD, DCR, pre-registered | Customize > Connectors > Add custom connector | Connect, or "Sign in when needed" | 3 |
| Codex CLI | yes | yes | `http_headers` in TOML only | DCR, CIMD, pre-registered | `codex mcp add <name> -- …` / `--url` | `codex mcp login <name>` | 1, else 2 |
| Codex app and IDE extension | yes | yes | TOML only | as Codex CLI | Settings > MCP servers > Add server (shares `~/.codex/config.toml`) | Authenticate | 1, else 2 |
| ChatGPT | – | yes | – (refused) | CIMD preferred, DCR | Plugins > + > Add custom MCP server (older: Settings > Apps > Developer mode > Create app) | OAuth on create | 3 |
| Gemini app | – | yes | – | DCR required; pre-registered | gemini.google.com Settings > Connected Apps > Custom apps > Add a custom app | Next | 3 |
| Gemini CLI | yes | yes (`httpUrl`) | `headers`, `-H` | DCR, pre-registered | `gemini mcp add -s user …` | automatic on 401, `/mcp auth <name>` | 1, else 2 |
| Antigravity app, IDE, `agy` | yes | yes (`serverUrl`) | `headers`, `agy mcp add -H` | pre-registered, CIMD, DCR | `agy mcp add <name> …`; `~/.gemini/config/mcp_config.json` | MCP Servers > Authenticate, paste the code; `/mcp` in `agy` | 1, else 2 |
| Kiro IDE and CLI | yes | yes | `headers` | DCR, CIMD, pre-registered | `kiro-cli mcp add --name … --scope global`; `~/.kiro/settings/mcp.json` | browser opens; `/mcp auth` | 1, else 2 |
| Cursor editor and CLI | yes | yes | `headers` (ignored when the server publishes OAuth metadata) | DCR, pre-registered; no CIMD | `~/.cursor/mcp.json`; install deeplink | editor prompt; `agent mcp login <name>` | 1, else 2 |
| VS Code with Copilot | yes | yes | `headers` | DCR, pre-registered; CIMD reported by third parties | `code --add-mcp '{…}'`; user `mcp.json` (`servers`) or `~/.copilot/mcp-config.json` | browser on first connection | 1, else 2 |
| GitHub Copilot CLI | yes | yes | `--header` | DCR, CIMD, pre-registered, device code | `copilot mcp add …`; `~/.copilot/mcp-config.json` | automatic; `/mcp auth <name>` | 1, else 2 |
| Devin Desktop (formerly Windsurf) | yes | yes | `headers` | DCR, pre-registered | `devin mcp add …`; `~/.config/devin/mcp_config.json` | `devin mcp login <name>`, Authenticate | 1, else 2 |
| Zed | yes | yes (`url`, since 0.214) | `headers` | standard MCP flow (DCR and CIMD per Zuplo) | `context_servers` in Zed settings | prompt when no `Authorization` header | 1, else 2 |
| JetBrains AI Assistant | yes | yes | ? | ? | Settings > Tools > AI Assistant > Model Context Protocol | ? | 1, else 4 |
| JetBrains Junie | yes | yes | `headers` | yes (method ?) | `~/.junie/mcp/mcp.json` | `/mcp` > Authorize | 1, else 2 |
| OpenCode | yes | yes | `headers`, `--header` | pre-registered, CIMD, DCR | `opencode mcp add`; `mcp.servers` in `~/.config/opencode/opencode.json` | `opencode mcp auth <name>`, `/mcps` | 1, else 2 |
| Cline | yes | yes (`type: streamableHttp`) | `headers` | DCR, pre-registered | `cline mcp add …`; `~/.cline/data/settings/cline_mcp_settings.json` | Authenticate after a 401 | 1, else 2 |
| Roo Code | yes | yes (`type: streamable-http`) | `headers` | – | Edit Global MCP (`mcp_settings.json`) | – | 1, else 4 |
| Kilo Code | yes | yes | `headers` | CIMD, DCR, pre-registered | `kilo mcp add …`; `~/.config/kilo/kilo.json` | `kilo mcp auth <name>` | 1, else 2 |
| Goose | yes | yes (`streamable_http`) | `headers` | CIMD, DCR, pre-registered | `~/.config/goose/config.yaml` `extensions`; `goose configure` | browser on connect | 1, else 2 |
| Continue | yes | yes (`type: streamable-http`) | `requestOptions.headers` | DCR | `~/.continue/config.yaml` `mcpServers` | Authenticate in Tools | 1, else 2 |
| Crush | yes | yes | `headers` | DCR, pre-registered (with `"oauth": true`) | `crushrc` or `crush.json` `mcp` | auth dialog | 1, else 2 |
| Qwen Code | yes | yes (`httpUrl`) | `headers` | DCR, pre-registered | `qwen mcp add --scope user …` | `/mcp` > Auth | 1, else 2 |
| Warp | yes | yes | `headers` | yes (method ?) | Settings > Agents > MCP servers; `~/.warp/.mcp.json` | browser on first start | 1, else 2 |
| Amp | yes | yes (auto-detected) | `--header` | CIMD, DCR, pre-registered | `amp mcp add <name> -- …` / `<url>` | when the TUI starts | 1, else 2 |
| Pi | yes, built in since 0.99 | yes | `--header` | CIMD, DCR, pre-registered | `pi mcp add <name> -- …` / `--url`; `~/.pi/agent/mcp.json` | `pi mcp login <name>`, `/mcp` | 1, else 2 |
| Factory Droid | yes | yes | `--header` | DCR, CIMD, pre-registered | `droid mcp add …`; `~/.factory/mcp.json` | `/mcp`, `droid mcp auth` | 1, else 2 |
| Augment (Auggie, IDE) | yes | yes | `-H` | DCR | `auggie mcp add …`; `~/.augment/settings.json` | ? | 1, else 2 |
| LM Studio | yes | yes | `headers` | ? | Program > Install > Edit `mcp.json` | ? | 1, else 4 |
| Jules | – | – | – | – | fixed list of partner servers | – | CLI in its VM |

Route "1, else 2" means: take the local stdio route when the agent can install the CLI, and the remote OAuth route
when it cannot. Route 4 applies only where OAuth is missing or undocumented and the CLI cannot be installed.

## What Ferrite's authorization server must hold for these clients

The Dashboard design advertises a registration endpoint, `client_id_metadata_document_supported: true`, S256, `none`
among token endpoint auth methods, and redirect URIs over `https`, loopback `http` on any port and private-use schemes.
Against the clients above:

- **Gemini app** needs a registration endpoint even when CIMD is offered, mandates S256, and requires the same refresh
  token back on refresh. The design keeps the refresh token for redirects on `oauth-redirect*.googleusercontent.com`.
- **Claude** chooses CIMD only when the server advertises both `client_id_metadata_document_supported: true` and
  `none`; the design advertises both. Its callback is `https://claude.ai/api/mcp/auth_callback`.
- **ChatGPT** registers by CIMD with `https://chatgpt.com/oauth/client.json` (or a per-callback variant) and redirects
  to `https://chatgpt.com/connector_platform_oauth_redirect` or `https://chatgpt.com/connector/oauth/{callback_id}`;
  the design's `fmcg` prefix keys on those.
- **Cursor** has no CIMD, so it always uses DCR, with the loopback redirect `http://localhost:8787/callback`. It
  ignores configured headers when the server publishes OAuth metadata, so the static-header route does not work in
  Cursor against Ferrite; the OAuth route does.
- **Antigravity** uses CIMD (`https://antigravity.google/oauth/client-metadata.json`) and the `https` redirect
  `https://antigravity.google/oauth-callback`, and asks the person to paste the code back; each of its three surfaces
  signs in separately. Open forum reports of failed token exchange on macOS were not rechecked.
- **VS Code** redirects to `http://127.0.0.1:33418` or `https://vscode.dev/redirect`; Kiro, Pi, Amp, Claude Code,
  Codex, Gemini CLI and Qwen Code use loopback redirects.

## Skills directories

Most agents now read Agent Skills from `~/.agents/skills/` as well as their own directory: Codex (whose current docs
name `~/.agents/skills` and older docs `~/.codex/skills`), Gemini CLI, Cursor (which also reads `~/.claude/skills` and
`~/.codex/skills`), GitHub Copilot CLI, Zed (only this one), Junie, OpenCode, Cline, Roo Code, Kilo Code, Goose, Amp,
Pi, Factory, Warp, Qwen Code, Crush and Devin. The ones that need their own directory are Claude Code
(`~/.claude/skills`), Antigravity (`~/.gemini/config/skills`), Kiro (`~/.kiro/skills`), VS Code Copilot
(`~/.copilot/skills`), Continue (`~/.continue/skills`) and Augment (`~/.augment/skills`). Several clients read more than
one of these, so installing the skill in two places can show it twice; the setup skill installs one copy and keeps a
working one it finds.

## Notes and changes this year

- **Gemini CLI** stopped serving Google AI Pro, Ultra and free users on 18 June 2026, and Antigravity CLI (`agy`)
  replaced it for them; it still serves paid API-key users and Gemini Code Assist Standard and Enterprise.
  [Google, 19 May 2026](https://developers.googleblog.com/en/an-important-update-transitioning-gemini-cli-to-antigravity-cli/)
- **Antigravity** keeps one MCP file for the app, IDE and CLI at `~/.gemini/config/mcp_config.json`; older guides name
  `~/.gemini/antigravity/mcp_config.json`. `agy mcp add [flags] <name> <commandOrUrl> [args…]` detects an `https` URL
  and writes it (installed `--help`). [docs](https://antigravity.google/docs/mcp),
  [skills](https://antigravity.google/docs/skills.md), [migration](https://antigravity.google/docs/cli/gcli-migration)
- **Windsurf** was renamed Devin Desktop on 2 June 2026, and its Cascade agent was replaced by Devin Local. Sources
  disagree on whether the old `~/.codeium/windsurf/mcp_config.json` (with `serverUrl`) is still read.
  [Devin Desktop MCP](https://docs.devin.ai/desktop/cascade/mcp), [CLI config](https://docs.devin.ai/cli/extensibility/mcp/configuration.md)
- **Pi** has built-in MCP since 0.99.0 (29 September 2026), reading `~/.pi/agent/mcp.json`; `pi-mcp-adapter` is now
  optional and turns the built-in client off. The package moved to `@earendil-works/pi-coding-agent`. Installed 1.5.1
  `--help` confirms `pi mcp add|list|login`.
- **OpenCode** moved servers under `mcp.servers.<name>`; the flat `mcp.<name>` form is marked deprecated in the
  published schema (`https://opencode.ai/config.json`, read 25 November) and still accepted. Installed 1.19.6 offers
  `opencode mcp add [name] --url --header` and `opencode mcp auth`.
- **VS Code** now recommends `~/.copilot/mcp-config.json`, shared with Copilot CLI; the profile `mcp.json` (with
  `servers` and `type`) is deprecated and due to be removed in February 2027.
  [reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration)
- **Roo Code** has no MCP OAuth, and its repository was archived in May 2026. **Kilo Code** is now built on OpenCode.
- **ChatGPT** documents full MCP (including writes) on the web for Business, Enterprise and Edu, read and fetch for
  Pro, refuses API keys and custom headers, and adds servers on the web only.
  [developer mode](https://developers.openai.com/api/docs/guides/developer-mode),
  [help](https://help.openai.com/en/articles/12584461), [auth](https://developers.openai.com/apps-sdk/build/auth)
- **Claude** custom connectors are on every plan (one on Free), sync from web or Desktop to the phone apps, and offer
  "Sign in when needed". Request headers are a limited beta.
  [add unlisted](https://claude.com/docs/connectors/custom/add-unlisted),
  [help](https://support.claude.com/en/articles/11176164)
- **Gemini app** custom apps are added on the web (a phone browser works) and then appear in the Android and iOS apps;
  the help page states 18 or older, US, personal account and English.
  [help](https://support.google.com/gemini/answer/17209137), [OAuth](https://developers.google.com/connected-apps/mcp/oauth)
- **Amazon Q Developer CLI** is now Kiro CLI. [AWS](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/upgrade-to-kiro.md)
- **The MCP specification** current revision is 2026-07-28. It deprecates DCR, orders client registration as
  pre-registered, then CIMD when advertised, then DCR, keeps RFC 9728 protected-resource discovery, the RFC 8707
  `resource` parameter and RFC 9207 `iss` validation, and makes the protocol stateless (no `initialize` or
  `Mcp-Session-Id`, a mandatory `server/discover`). Ferrite's `/mcp` has not been checked against a client that speaks
  only 2026-07-28. [versioning](https://modelcontextprotocol.io/specification/versioning),
  [client registration](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration.md),
  [changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog.md)

## Sources by client

- Claude Code: [MCP](https://code.claude.com/docs/en/mcp), [skills](https://code.claude.com/docs/en/skills)
- Claude Desktop: [local servers](https://modelcontextprotocol.io/docs/develop/connect-local-servers),
  [extensions](https://support.claude.com/en/articles/10949351)
- Codex: [MCP](https://learn.chatgpt.com/docs/extend/mcp?surface=cli),
  [config reference](https://learn.chatgpt.com/docs/config-file/config-reference),
  [app](https://learn.chatgpt.com/docs/extend/mcp?surface=app), [skills](https://learn.chatgpt.com/docs/build-skills)
- Gemini CLI: [MCP](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/tools/mcp-server.md),
  [skills](https://geminicli.com/docs/cli/skills/)
- Kiro: [configuration](https://kiro.dev/docs/mcp/configuration/), [CLI](https://kiro.dev/docs/reference/cli-commands/),
  [skills](https://kiro.dev/docs/skills/)
- Cursor: [MCP](https://cursor.com/docs/context/mcp), [install links](https://cursor.com/docs/context/mcp/install-links),
  [CLI](https://cursor.com/docs/cli/mcp), [skills](https://cursor.com/docs/context/skills),
  [headers ignored with OAuth](https://forum.cursor.com/t/mcp-headers-config-ignored-when-server-has-oauth-discovery/156054),
  [no CIMD](https://forum.cursor.com/t/mcp-oauth-cimd-support-plans-and-timelines/148096)
- VS Code: [MCP guide](https://code.visualstudio.com/api/extension-guides/ai/mcp),
  [skills](https://code.visualstudio.com/docs/agent-customization/agent-skills.md)
- Copilot CLI: [add servers](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers),
  [reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference),
  [skills](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
- Zed: [MCP](https://zed.dev/docs/ai/mcp), [0.214.0](https://zed.dev/releases/preview/0.214.0),
  [skills](https://zed.dev/docs/ai/skills), [Zuplo matrix](https://zuplo.com/learn/mcp/compatibility/clients/zed.md)
- JetBrains: [AI Assistant MCP](https://www.jetbrains.com/help/ai-assistant/mcp.html),
  [Junie MCP](https://junie.jetbrains.com/docs/junie-cli-mcp-configuration.html),
  [Junie skills](https://junie.jetbrains.com/docs/agent-skills.html)
- OpenCode: [config](https://opencode.ai/docs/config), [MCP servers](https://opencode.ai/docs/mcp-servers),
  [skills](https://opencode.ai/docs/skills)
- Cline: [remote servers](https://docs.cline.bot/mcp/connecting-to-a-remote-server.md)
- Roo Code: [MCP](https://roocodeinc.github.io/Roo-Code/features/mcp/using-mcp-in-roo)
- Kilo Code: [MCP](https://kilo.ai/docs/features/mcp/using-mcp-in-kilo-code)
- Goose: [config files](https://goose-docs.ai/docs/guides/config-files),
  [extensions](https://goose-docs.ai/docs/getting-started/using-extensions)
- Continue: [MCP](https://docs.continue.dev/customize/deep-dives/mcp)
- Qwen Code: [MCP](https://qwenlm.github.io/qwen-code-docs/en/users/features/mcp/)
- Warp: [MCP](https://docs.warp.dev/agents/capabilities/mcp.md), [skills](https://docs.warp.dev/agents/capabilities/skills.md)
- Amp: [MCP](https://ampcode.com/docs/customize/mcp), [skills](https://ampcode.com/docs/customize/skills)
- Pi: [package](https://www.npmjs.com/package/@earendil-works/pi-coding-agent), its bundled `docs/mcp.md`
- Factory: [MCP](https://docs.factory.com/cli/configuration/mcp), [skills](https://docs.factory.com/cli/configuration/skills)
- Augment: [CLI integrations](https://docs.augmentcode.com/cli/integrations), [IDE](https://docs.augmentcode.com/setup-augment/mcp)
- LM Studio: [MCP](https://lmstudio.ai/docs/app/mcp)
- Jules: [changelog 2 February 2026](https://jules.google/docs/changelog/2026-02-02)

## Not verified

None of these clients was run against Ferrite's authorization server for this record. Unverified points: whether
Claude Desktop's config file accepts a remote URL; whether a ChatGPT plugin created on the web works in its phone and
desktop apps, and its availability on Plus and Free; which Codex skills directory wins when both exist; the Gemini
app's plan requirement; how Warp, Junie and Augment register an OAuth client; whether JetBrains AI Assistant sends
headers or does OAuth; the Roo Code global settings path; and the current state of Antigravity's open OAuth bugs.
