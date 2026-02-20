# KoKo Finance MCP Tool Guide

Detailed parameter and response reference for each tool. All tools are read-only
and idempotent unless noted.

## search_credit_cards

Search for credit cards using natural language.

**Parameters:**
- `query` (string, required): Natural language search, e.g. "best travel card under $200 annual fee"
- `max_results` (int, optional, default 5): Number of results (1-10)

**Returns:**
```json
{
  "success": true,
  "query": "...",
  "recommendations": [
    {
      "card_name": "Chase Sapphire Preferred",
      "issuer": "Chase",
      "annual_fee": 95,
      "why_recommended": "...",
      "fit_score": 92,
      "apply_url": "https://...",
      "key_benefits": "..."
    }
  ],
  "total_found": 15
}
```

**Tips:**
- Include specific criteria in the query for better results: fee limits, credit score, spending categories
- Use `fit_score` to rank relevance (0-100)

## compare_cards

Compare 2-3 credit cards side by side with AI analysis.

**Parameters:**
- `card_names` (list of strings, required): 2-3 card names, e.g. `["Chase Sapphire Preferred", "Amex Gold"]`
- `monthly_spending` (dict, optional): Spending by category, e.g. `{"dining": 500, "travel": 300}`

**Returns:**
```json
{
  "success": true,
  "cards_compared": ["Chase Sapphire Preferred", "Amex Gold"],
  "comparison": {
    "analysis": "...",
    "winner": "...",
    "use_cases": "..."
  }
}
```

**Tips:**
- Always provide `monthly_spending` when available — it personalizes the winner recommendation
- If a card isn't found, check `not_found_cards` in the response

## get_card_details

Get full details for a specific card.

**Parameters:**
- `card_name` (string, required): Card name (partial match OK), e.g. "Sapphire Preferred"
- `issuer` (string, optional): Narrow by issuer, e.g. "Chase"

**Returns:**
```json
{
  "success": true,
  "card": {
    "card_name": "Chase Sapphire Preferred Card",
    "issuer": "Chase",
    "network": "Visa",
    "card_type": "travel",
    "annual_fee": 95,
    "credit_score_needed": "670+",
    "rewards_structure": "3x dining, 2x travel, 1x everything else",
    "key_benefits": "...",
    "welcome_bonus": "...",
    "sign_on_bonus": 60000,
    "sign_on_bonus_type": "points",
    "foreign_transaction_fee": "None",
    "apply_url": "https://...",
    "best_for": "...",
    "partner_brand": null
  }
}
```

**Tips:**
- Common abbreviations work: "Amex Gold", "CSP", "Venture X"
- If `card` is null, the card wasn't found — suggest the user check spelling

## calculate_card_value

Calculate whether a card's annual fee is worth it.

**Parameters:**
- `annual_spending` (float, required): Total annual spend on this card
- `annual_fee` (float, required): Card's annual fee
- `sign_on_bonus` (float, optional, default 0): Bonus amount
- `sign_on_bonus_type` (string, optional, default "points"): "points", "cash", or "multiplier"

**Returns:**
```json
{
  "success": true,
  "first_year": {
    "total_value": 850.00,
    "net_value": 755.00,
    "base_rewards": 200.00,
    "sign_on_bonus_value": 600.00,
    "benefits_value": 50.00
  },
  "ongoing_annual": {
    "total_value": 250.00,
    "net_value": 155.00
  },
  "breakeven_annual_spend": 9500.00,
  "verdict": "Estimated rewards and credits exceed the annual fee",
  "note": "Estimates assume typical benefit utilization and standard point redemption values. Actual value depends on spending patterns and how you redeem rewards."
}
```

**Tips:**
- Use first_year values when the user is considering a new card (includes sign-on bonus)
- Use ongoing_annual values for existing cards
- The `breakeven_annual_spend` tells users the approximate spend needed to offset the fee
- **Important:** `net_value` fields are estimates, not cash. Present as "estimated
  rewards and credits compared to the annual fee," never as money earned or saved

## optimize_portfolio

Analyze a full card portfolio with health scoring and verdicts.

**Parameters:**
- `card_names` (list of strings, required): All cards the user owns
- `monthly_spending` (dict, optional): Spending by category
- `session_id` (string, optional): From a previous call for continuity

**Returns:**
```json
{
  "success": true,
  "health_score": 78,
  "net_annual_value": 425.00,
  "total_annual_fees": 690,
  "card_details": [
    {
      "card_name": "Chase Sapphire Reserve",
      "annual_fee": 550,
      "net_value": 312.00,
      "verdict": "KEEP",
      "reason": "High travel rewards offset the fee"
    }
  ],
  "strategies": ["...", "...", "..."],
  "quick_wins": ["...", "..."],
  "session_id": "mcp_abc123"
}
```

**Tips:**
- Save the `session_id` for follow-up calls in the same conversation
- Verdicts are KEEP, OPTIMIZE, or CANCEL — present in bold
- `health_score` interpretation: 80+ excellent, 60-79 good, 40-59 fair, below 40 needs attention
- **Important:** `net_annual_value` and per-card `net_value` are estimates. Present as
  "estimated rewards and credits [exceed/fall short of] total fees by ~$X," not as earnings

## recommend_card_for_category

Find the best card for a specific spending category.

**Parameters:**
- `card_names` (list of strings, required): Cards the user owns
- `category` (string, required): One of: `groceries`, `dining`, `travel`, `gas`, `online_shopping`, `everything_else`
- `amount` (float, optional, default 100): Purchase amount
- `session_id` (string, optional): For continuity

**Returns:**
```json
{
  "success": true,
  "category": "dining",
  "recommended_card": "American Express Gold Card",
  "issuer": "American Express",
  "headline": "4x points on dining — your best option",
  "explanation": "...",
  "pro_tip": "...",
  "alternatives": [
    {
      "card_name": "Chase Sapphire Reserve",
      "issuer": "Chase",
      "value_summary": "3x on dining",
      "why_use": "Better if you need Visa acceptance",
      "pro_tip": "..."
    }
  ],
  "bottom_line": "..."
}
```

**Tips:**
- Call this multiple times with different categories to build a full "wallet strategy"
- The `alternatives` array shows other good options with trade-offs
- `pro_tip` often contains non-obvious advice worth highlighting

## create_mcp_session

Create a session for tracking analysis across multiple calls.

**Parameters:** None required.

**Returns:**
```json
{
  "success": true,
  "session_id": "mcp_abc123def456",
  "message": "Session created. Pass this session_id to other tools..."
}
```

**Tips:**
- Only needed if the user will make multiple portfolio-related queries
- `optimize_portfolio` and `recommend_card_for_category` auto-create sessions if none provided
- Pass the returned `session_id` to subsequent tool calls for continuity
