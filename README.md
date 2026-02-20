# KoKo Credit Card Advisor — Claude Code Plugin

A Claude Code plugin that gives Claude expert knowledge about credit cards. Connects to the [KoKo Finance](https://kokofinance.net) MCP server for real-time card data and provides a skill that teaches Claude how to run credit card workflows.

## What's Included

- **MCP Server Connection** — Auto-connects to `kokofinance.net/mcp/` with 7 tools for searching, comparing, and analyzing credit cards
- **Credit Card Advisor Skill** — Teaches Claude 5 workflow patterns: portfolio review, which-card-to-use, new card finder, travel planning, and large purchase planning

## Installation

### From Claude Code CLI

```bash
# Clone and install
git clone https://github.com/madanc/koko-credit-card-plugin.git
claude --plugin-dir ./koko-credit-card-plugin
```

### Manual Setup

Copy the plugin directory to your Claude Code plugins location, or point to it with `--plugin-dir`.

## Usage Examples

### Review Your Card Portfolio

> "I have a Chase Sapphire Reserve, Amex Gold, and Citi Double Cash. Review my portfolio and tell me which cards are worth keeping."

Claude will analyze each card's net value, give KEEP/OPTIMIZE/CANCEL verdicts, and suggest optimization strategies.

### Which Card Should I Use?

> "I'm about to spend $200 on groceries. I have an Amex Gold, Chase Freedom Unlimited, and a Citi Custom Cash. Which card should I use?"

Claude will compare rewards rates across your cards for that category and recommend the best one with a pro tip.

### Find a New Card

> "I spend a lot on travel and dining. I want a card with no more than $250 annual fee. What do you recommend?"

Claude will search the card database, compare the top matches side by side, and give a clear recommendation with an application link.

## Available MCP Tools

| Tool | Description |
|------|------------|
| `search_credit_cards` | Natural language card search |
| `compare_cards` | Side-by-side comparison of 2-3 cards |
| `get_card_details` | Full details for a specific card |
| `calculate_card_value` | Annual fee break-even analysis |
| `optimize_portfolio` | Portfolio health score and verdicts |
| `recommend_card_for_category` | Best card for a spending category |
| `create_mcp_session` | Session tracking across multiple queries |

## Links

- [KoKo Finance](https://kokofinance.net) — Main site
- [Developer Docs](https://kokofinance.net/developers.html) — MCP server documentation
- [MCP Endpoint](https://kokofinance.net/mcp/) — Direct MCP server URL

## License

MIT
