# KoKo Credit Card Advisor — Claude Code Plugin

Credit card intelligence for Claude. Find and compare cards, optimize rewards. Covers break-even calculations, renewal questions, and merchant intelligence. Connects to the [KoKo Finance](https://kokofinance.net) MCP server for real-time card data and provides a skill that teaches Claude how to run these workflows.

## What's Included

- **MCP Server Connection** — Auto-connects to `kokofinance.net/mcp/` with 14 tools and 5 prompts for searching, comparing, and analyzing credit cards
- **Credit Card Advisor Skill** — Teaches Claude 10 workflow patterns: portfolio review, which-card-to-use, new card finder, travel planning, large purchase planning, card renewal decisions, merchant-specific advice, benefits deep dive, card terms and cost of carry, and card change monitoring

## Installation

### From Claude Code CLI

```bash
# Clone and install
git clone https://github.com/KokoFinance/koko-credit-card-plugin.git
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

### Should I Renew This Card?

> "I only use the Uber credit and airline fee credit on my Amex Platinum. Is it still worth paying the $695 annual fee?"

Claude will weigh year-2+ rewards and benefits against the fee and give a RENEW / DOWNGRADE / CANCEL_AND_REPLACE verdict.

### Which Card at This Merchant?

> "I'm buying coffee at Starbucks. I have a Chase Sapphire Reserve, Amex Gold, and Citi Double Cash. Which card earns the most?"

Claude auto-detects the spending category for the merchant and ranks your cards by reward value.

## Available MCP Tools

| Tool | Description |
|------|------------|
| `search_credit_cards` | Natural language card search |
| `compare_cards` | Side-by-side comparison of 2-3 cards |
| `get_card_details` | Full details for a specific card |
| `calculate_card_value` | Annual fee break-even analysis |
| `optimize_portfolio` | Portfolio health score and verdicts |
| `recommend_card_for_category` | Best card for a spending category |
| `check_card_renewal` | RENEW/DOWNGRADE/CANCEL_AND_REPLACE verdict for a card up for renewal |
| `which_card_at_merchant` | Best card from your portfolio at a specific merchant |
| `check_merchant_benefits` | Check if any cards have credits at a merchant |
| `get_card_benefits` | All credits/benefits for a card |
| `get_card_terms` | APR, penalty fees, and other Schumer Box terms |
| `get_card_changes` | Audit log of recent card data changes |
| `get_program_trends` | Points program valuation history (CPP, transfer ratios) |
| `create_mcp_session` | Session tracking across multiple queries |

Several tools also accept optional personalization parameters (`credit_tier`, `primary_goal`, `issuer_preferences`, `spending`, and custom KEEP/CANCEL thresholds) — see the [tool guide](skills/credit-card-advisor/references/tool-guide.md) for the full parameter reference, or the [MCP server docs](https://github.com/KokoFinance/koko-mcp-server) for details shared across all MCP clients.

## Links

- [KoKo Finance](https://kokofinance.net) — Main site
- [Developer Docs](https://kokofinance.net/developers.html) — MCP server documentation
- [MCP Endpoint](https://kokofinance.net/mcp/) — Direct MCP server URL
- [KoKo Finance MCP Server](https://github.com/KokoFinance/koko-mcp-server) — Standalone MCP server repo for non-Claude-Code clients

## License

MIT
