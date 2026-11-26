# Ferrite agent client matrix

A coding agent or chat app can reach Ferrite in one of four ways, and which one works depends on the client: whether it can run a local server, whether it speaks remote MCP, how it signs in, and where it keeps its configuration. This repository records that, client by client.

[`docs/matrix.md`](docs/matrix.md) is the matrix. It has two parts. The first records runs against Ferrite Cloud: each agent was given one prompt in a clean environment and asked to set itself up, build a small poll application and deploy it, and the table gives the outcome, the time taken and what each client did when asked to sign in to the hosted server. The second is the research record: for each client, the transports it supports, how it registers for OAuth, the command or settings page that adds a server, what triggers sign-in, and the route Ferrite's setup instructions take as a result. It also lists what an authorization server has to accept for these clients, where each one reads Agent Skills from, and the changes to the clients during 2026.

## The four routes

1. **Local stdio.** The agent can run a terminal and install software, so it installs the `ferrite` CLI and registers `ferrite mcp-serve` as a local server.
2. **Remote with OAuth.** The agent adds `https://mcp.ferrite.cloud/mcp` to its own configuration and signs in through the browser.
3. **Connector.** In a chat app, the person adds the same address once in the app's settings.
4. **Static header.** A client with neither OAuth nor a usable local server sends a fixed `Authorization` header.

## How the matrix was compiled

The runs were made on 26 November 2026. Each one started from an empty home directory and an empty project, with no Ferrite software, credentials or configuration present, and carried in only the agent's own model login. A run passes when the agent returns an `https` address and a separate request to that address gets the page, posts a vote and sees the count change. The document names the agents that could not be run and says why.

The research record is dated 25 November 2026. Its facts come from each vendor's current documentation, which the document links, and for seven clients the commands were also read from the installed program's own `--help`. It says where a source was silent ("?") and where two sources disagree, and the runs section lists the points where a run contradicted it.

The transcripts of the runs, and the harness that will repeat them for every client, are not in this repository.

Corrections are welcome as issues, especially where a client's behaviour has changed since the date above. Please include the client's version and a link to the documentation or the command output that shows the change.

## Licence and contributions

The text is copyright iGent AI and is not offered under an open source licence; see `LICENSE`. This repository accepts issues and does not accept outside pull requests; see `CONTRIBUTING.md`. Report a security problem to security@ferrite.cloud, as `SECURITY.md` describes. The hosted Ferrite platform is proprietary and not open source.

This repository is a scrubbed public snapshot prepared by the Ferrite team and carries no private development history.
