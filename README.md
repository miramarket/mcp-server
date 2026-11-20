# Miramarket MCP server

The public connector card for **Miramarket**'s hosted MCP server. Miramarket is a trading account for
three kinds of market from one place: prediction markets on Polymarket, perpetual futures on
Hyperliquid (crypto, stocks, indices, commodities, FX, rates), and tokenized stocks on Base.

An AI assistant connected to this server can look up markets, build a strategy from a plain-English
request, simulate it, and — only if you allow it, and only up to a budget you set — start a live run
with real money.

- **Endpoint:** `https://api.miramarket.org/mcp`
- **Transport:** Streamable HTTP (every answer is an SSE stream, with a keepalive frame every 10s)
- **Auth:** OAuth 2.1 with PKCE (S256), dynamic client registration or a Client ID Metadata Document
- **Docs:** [Connect an AI assistant](https://docs.miramarket.org/mcp-server) ·
  [MCP reference](https://docs.miramarket.org/mcp-reference) ·
  [Machine-readable reference](https://docs.miramarket.org/mcp-reference.md)
- **Site:** https://www.miramarket.org

The server itself is hosted and closed-source; this repository holds the connection details,
`server.json` for the MCP Registry, and the tool reference so directories and clients have a public
project page to point at.

## Connect

```bash
# Claude Code
claude mcp add --transport http miramarket https://api.miramarket.org/mcp
```

Then type `/mcp` in Claude Code and choose **miramarket**; your browser opens the Miramarket approval
page.

**Claude on the web, desktop and mobile, ChatGPT (developer mode) and Grok** each take the same
address as a custom connector. Any other client that speaks remote MCP over Streamable HTTP with
OAuth sign-in connects the same way.

```jsonc
// HTTP clients such as Cursor, LibreChat, Open WebUI, FinceptTerminal
{
  "mcpServers": {
    "miramarket": {
      "type": "http",
      "url": "https://api.miramarket.org/mcp"
    }
  }
}
```

Server metadata for clients that discover authorization:

- Resource metadata: `https://api.miramarket.org/.well-known/oauth-protected-resource/mcp`
- Authorization server: `https://api.miramarket.org/.well-known/oauth-authorization-server`
- Resource to request tokens for: `https://api.miramarket.org/mcp`

## Scopes

| Scope | Allows |
| - | - |
| `account:read` | Always granted. Read strategies, runs and committed funds. |
| `strategies:write` | Build, validate and save strategies; start and stop simulations. |
| `trade` | Start live runs within the connection's budget, ask the owner to approve ones beyond it, and stop live runs. |

There is no withdrawal scope, and the connection cannot place single orders or bridge funds. The
account owner chooses the scopes and the budget on the approval page and can revoke the connection at
any time from **Connected agents** in the app.

## Tools

| Tool | What it does | Scope |
| - | - | - |
| `whoami` | This connection's scopes, live budget, how much of it is in use. | — |
| `capital_summary` | Money committed to running strategies and pending withdrawals. | `account:read` |
| `strategies_list` | The account's strategies, newest first, with status and money. | `account:read` |
| `strategy_show` | One strategy's nodes, principal, latest run and app link. | `account:read` |
| `markets_search` | Open Polymarket markets by keyword, with prices, liquidity and volume. | public |
| `markets_by_tag` | Open Polymarket markets under a tag slug, most traded first. | public |
| `perps_list` | Hyperliquid perps by category, with mark price, funding and max leverage. | public |
| `perps_search` | Hyperliquid perps by name or ticker. | public |
| `stocks_list` | Tokenized stocks on Base, with swap price and the price a condition watches. | public |
| `strategy_guide` | How to design a strategy, write its request, or write its JSON. | — |
| `strategy_generate` | Builds a strategy from a description with the same assistant as the app. | `strategies:write` |
| `strategy_generate_status` | Follows a build that answered `working`. | `strategies:write` |
| `strategy_validate` | Checks a strategy JSON with the same validator the app uses. | — |
| `strategy_post` | Saves a strategy as a draft or starts a simulation. No real money moves. | `strategies:write` |
| `strategy_start_live` | Starts a saved draft with real money, within the budget or with the owner's approval. | `trade` |
| `strategy_live_approval` | Where a live start waiting for the owner stands. | — |
| `strategy_cancel` | Stops a run; a live run's open positions close at market. | `trade` / `strategies:write` |
| `withdraw_in_app` | Points the owner to the app, because agents cannot withdraw. | — |

A strategy the assistant builds is shown to you leg by leg — venue, market, side, amount, leverage —
before anything happens. In hosts that render MCP Apps (Claude, ChatGPT) it arrives as an inline
card with **Simulate** and **Run Live** buttons; elsewhere it arrives as a summary and a link that
opens the strategy in the Miramarket builder.

## Safety boundary

- The agent never withdraws. Withdrawals happen in the app, by the account owner.
- A live run above the connection's budget waits for the owner on **Connected agents**
  (`app.miramarket.org/settings/agents`) and expires after a day.
- Only the owner can set or change the budget; the assistant cannot.
- Claude Code asks before every live start and every stop. Other hosts have their own confirmation
  settings.
- Simulation (`mode: "demo"`) uses no real funds.

Trading involves risk and leverage magnifies losses. Execution and fills are not guaranteed. Nothing
here is financial, investment, legal or tax advice.

## Registry

`server.json` publishes this server to the official MCP Registry as
`io.github.miramarket/miramarket`:

```bash
mcp-publisher validate
mcp-publisher publish
```

## License

Documentation in this repository may be read and quoted freely. Miramarket itself is proprietary —
the hosted server, the app and the strategy engine are not distributed here.
