# KoKo Finance MCP Tool Guide

Detailed parameter and response reference for each tool. All tools are read-only
and idempotent unless noted.

## search_credit_cards

Search for credit cards using natural language.

**Parameters:**
- `query` (string, required): Natural language search, e.g. "best travel card under $200 annual fee"
- `max_results` (int, optional, default 5): Number of results (1-10)
- `card_type` (string, optional): Filter by card type, e.g. "travel", "cashback", "balance_transfer", "business", "hotel", "airline"
- `issuer` (string, optional): Filter by issuer, e.g. "Chase", "Amex", "Capital One", "Citi"
- `max_annual_fee` (int, optional): Maximum annual fee in dollars — cards above this are filtered out
- `credit_tier` (string, optional): Filter by credit tier eligibility — one of `"poor"`, `"fair"`, `"good"`, `"excellent"`

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
- `credit_tier` (string, optional): User's credit tier for context — one of `"poor"`, `"fair"`, `"good"`, `"excellent"`
- `primary_goal` (string, optional): User's primary goal for context — one of `"travel"`, `"cashback"`, `"build_credit"`, `"business"`, `"balance_transfer"`
- `issuer_preferences` (list of dicts, optional): Boost/de-boost specific issuers, e.g. `[{"issuer": "Chase", "weight": 1.5}]` (1.0 = neutral, range 0.1-3.0)

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
    "partner_brand": null,
    "points_program_key": "chase_ur",
    "portal_cpp": 1.5
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
- `card_name` (string, optional): When provided, uses the card's actual benefits and category earning multipliers from the database instead of conservative default rates
- `monthly_spending` (dict, optional): Spending by category, e.g. `{"groceries": 500, "dining": 300, "travel": 200}` — used with `card_name` for category-aware reward calculations
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
- `point_balances` (list of dicts, optional): Don't ask upfront. Each item: `{"program": "chase_ur", "balance": 70000}` — warns about stranded points on a CANCEL verdict
- `benefit_selections` (list of strings, optional): Don't ask upfront. Keys: `uber`, `dining`, `travel`, `airline_fee`, `hotel_fhr`, `saks`, `clear_plus`, `digital_entertainment`, `admirals_club`, `global_entry`. Selected count at 100% utilization, unselected at 0%
- `session_id` (string, optional): From a previous call for continuity
- `credit_tier` (string, optional): One of `"poor"`, `"fair"`, `"good"`, `"excellent"`
- `primary_goal` (string, optional): One of `"travel"`, `"cashback"`, `"build_credit"`, `"business"`, `"balance_transfer"`
- `issuer_preferences` (list of dicts, optional): e.g. `[{"issuer": "Chase", "weight": 1.5}]` (1.0 = neutral, range 0.1-3.0)
- `analysis_type` (string, optional): Focus the analysis — one of `"general"`, `"fees"`, `"benefits"`, `"usage"`
- `cancel_threshold` (float, optional, default -200): Cards with net value below this get CANCEL
- `keep_threshold` (float, optional, default 100): Cards with net value above this get KEEP

**Returns:**
```json
{
  "success": true,
  "health_score": 78,
  "net_annual_value": 425.00,
  "net_annual_value_display": "$425",
  "total_annual_fees": 690,
  "total_annual_fees_display": "$690",
  "card_details": [
    {
      "card_name": "Chase Sapphire Reserve",
      "annual_fee": 550,
      "annual_fee_display": "$550",
      "net_value": 312.00,
      "net_value_display": "$312",
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
- **Critical:** always use `annual_fee_display`, `net_value_display`, `total_annual_fees_display`,
  and `net_annual_value_display` verbatim in tables — never retype a dollar amount from the raw
  numeric field. See "Never retype dollar amounts" in SKILL.md.

## recommend_card_for_category

Find the best card for a specific spending category.

**Parameters:**
- `card_names` (list of strings, required): Cards the user owns
- `category` (string, required): One of: `groceries`, `dining`, `travel`, `gas`, `online_shopping`, `car_rental`, `everything_else`
- `amount` (float, optional, default 100): Purchase amount
- `spending` (dict, optional): Full monthly spending profile in addition to `amount`, e.g. `{"groceries": 500, "dining": 300}` — gives fuller context for the recommendation
- `credit_tier` (string, optional): One of `"poor"`, `"fair"`, `"good"`, `"excellent"`
- `primary_goal` (string, optional): One of `"travel"`, `"cashback"`, `"build_credit"`, `"business"`, `"balance_transfer"`
- `issuer_preferences` (list of dicts, optional): e.g. `[{"issuer": "Chase", "weight": 1.5}]` (1.0 = neutral, range 0.1-3.0)
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
- For `car_rental`, the recommendation weighs insurance benefits (CDW, primary vs secondary) alongside rewards — not just points earned

## check_card_renewal

Check whether a card is worth renewing when its annual fee comes due.

Analyzes the card's year-2+ value (without the sign-up bonus), finds same-issuer
downgrade options, and suggests replacement cards if canceling makes sense. Ask
the user for their monthly spending before calling this tool — a casual question
is fine. Do NOT ask about benefits upfront; only pass `benefit_selections` if the
user volunteers them after seeing initial results.

**Parameters:**
- `card_name` (string, required): The card to evaluate, e.g. "Chase Sapphire Reserve"
- `monthly_spending` (dict, optional): Spending by category, e.g. `{"groceries": 500, "dining": 300}`, or a single total `{"total": 5000}`. Omit to use national averages.
- `other_cards` (list of strings, optional): Other cards in the user's wallet for portfolio context
- `benefit_selections` (list of strings, optional): Don't ask upfront. Keys: `uber`, `dining`, `travel`, `airline_fee`, `hotel_fhr`, `saks`, `clear_plus`, `digital_entertainment`, `admirals_club`, `global_entry`
- `benefit_categories_used` (list of strings, optional): Coarser alternative to `benefit_selections`. Valid: `dining_credit`, `travel_credit`, `hotel_credit`, `lounge_access`, `shopping_credit`, `entertainment_credit`, `business_credit`, `loyalty_bonus`, `other`
- `acquired_date` (string, optional): `YYYY-MM-DD` — used to calculate card tenure
- `credit_tier` (string, optional): One of `"poor"`, `"fair"`, `"good"`, `"excellent"`
- `primary_goal` (string, optional): One of `"travel"`, `"cashback"`, `"build_credit"`, `"business"`, `"balance_transfer"`
- `renewal_cancel_threshold` (float, optional, default -100): Net value below this → CANCEL_AND_REPLACE
- `renewal_keep_threshold` (float, optional, default 50): Net value at or above this → RENEW
- `session_id` (string, optional): From a previous call

**Returns:**
```json
{
  "success": true,
  "verdict": "RENEW",
  "verdict_reason": "...",
  "confidence": "high",
  "annual_fee": 550,
  "year2_rewards_value": 380.00,
  "year2_benefits_value": 300.00,
  "year2_total_value": 680.00,
  "year2_net_value": 130.00,
  "breakeven_monthly_spend": 916.67,
  "downgrade_options": [],
  "replacement_options": [],
  "detailed_analysis": "...",
  "retention_tips": ["...", "..."],
  "timing_guidance": "...",
  "card_name": "Chase Sapphire Reserve",
  "issuer": "Chase",
  "spending_source": "user_provided",
  "spending_data_quality": "...",
  "thresholds_used": {"cancel": -100, "keep": 50}
}
```

**Tips:**
- Verdicts are RENEW, DOWNGRADE, or CANCEL_AND_REPLACE
- `downgrade_options` (same-issuer, lower fee) and `replacement_options` (different-issuer,
  matched to the user's spending) are both computed on every call regardless of verdict —
  check both, don't assume one is empty just because the verdict doesn't match its "classic"
  case. For a DOWNGRADE verdict specifically, present both as two distinct paths rather than
  only the downgrade — see "Card Renewal Decision" in SKILL.md for how to frame this.
- `replacement_options` is genuinely empty only when `downgrade_options` is also empty (that's
  what makes the verdict CANCEL_AND_REPLACE instead of DOWNGRADE in the first place)
- `year2_*` fields exclude the sign-up bonus — they reflect ongoing, steady-state value
- **Important:** `year2_net_value` is an estimate. Present as "estimated rewards and credits [exceed/fall short of] the annual fee by ~$X," not as earnings

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

## which_card_at_merchant

Find the best card to use at a specific merchant. Auto-detects the spending category.

**Parameters:**
- `merchant` (string, required): Merchant name, e.g. "Starbucks", "Saks Fifth Avenue", "Shell", "Delta Air Lines"
- `card_names` (list of strings, required): Cards the user owns
- `amount` (float, optional, default 100): Purchase amount

**Returns:**
```json
{
  "success": true,
  "recommended_card": "American Express Gold Card",
  "category_detected": "dining",
  "category_method": "gemini",
  "reason": "Starbucks codes as dining — Amex Gold 4x vs Chase Sapphire Reserve 3x",
  "earnings_comparison": [
    {"card": "American Express Gold Card", "multiplier": "4x", "earnings": 4.00},
    {"card": "Chase Sapphire Reserve", "multiplier": "3x", "earnings": 3.00}
  ]
}
```

**Tips:**
- Unlike `recommend_card_for_category`, the user doesn't need to know the spending category — just name the merchant
- The `earnings_comparison` array ranks all cards by reward value at that merchant
- `category_method` shows how the category was resolved (e.g. "gemini" for AI-resolved)

## check_merchant_benefits

Check if any cards have credits or benefits at a specific merchant.

**Parameters:**
- `merchant` (string, required): Merchant name, e.g. "Saks Fifth Avenue", "Uber", "Disney+", "Delta Air Lines"
- `card_names` (list of strings, required): Cards the user owns

**Returns:**
```json
{
  "success": true,
  "matching_benefits": [
    {
      "card": "American Express Platinum Card",
      "benefit": "Saks Fifth Avenue Credit",
      "value": 100,
      "frequency": "semi-annual"
    }
  ],
  "earning_recommendation": {
    "best_card": "American Express Platinum Card",
    "multiplier": "1x",
    "note": "Saks codes as general — Amex Platinum earns 1x. Use Amex Platinum to combine with the credit."
  }
}
```

**Tips:**
- Pair with `which_card_at_merchant` to give complete merchant advice (credits + earning)
- When the best-earning card differs from the card with credits, the `note` explains the trade-off
- `matching_benefits` is empty if no cards have credits at that merchant

## get_card_benefits

Get all credits, benefits, and rewards multipliers for a specific card.

**Parameters:**
- `card_name` (string, required): Card name, e.g. "Amex Platinum", "Chase Sapphire Reserve"

**Returns:**
```json
{
  "success": true,
  "card": "American Express Platinum Card",
  "issuer": "American Express",
  "annual_fee": 695,
  "credits": [
    {
      "name": "Uber Credit",
      "value": 200,
      "frequency": "monthly",
      "schedule": "$15/month + $20 December bonus",
      "conditions": "Enrolled Amex Platinum cardmembers"
    }
  ],
  "total_credit_value": 1400,
  "rewards_multipliers": {"flights": 5, "hotels": 5, "dining": 1, "general": 1},
  "points_program": "amex_mr",
  "portal_cpp": 1.0
}
```

**Tips:**
- More detailed than `get_card_details` — includes individual credit breakdowns with frequency and conditions
- Compare `total_credit_value` against `annual_fee` to show how credits offset the fee
- Use this when the user asks specifically about benefits, not general card info

## get_card_terms

Get Schumer Box data — APR, penalties, and fee terms for a card.

**Parameters:**
- `card_name` (string, required): Card name (fuzzy matching applied)
- `issuer` (string, optional): Narrow by issuer, e.g. "Chase"

**Returns:**
```json
{
  "success": true,
  "card": "Chase Sapphire Reserve",
  "issuer": "Chase",
  "terms": {
    "purchase_apr": "22.49% - 29.49%",
    "cash_advance_apr": "29.49%",
    "penalty_apr": "Up to 29.99%",
    "balance_transfer_apr": "22.49% - 29.49%",
    "late_fee": "Up to $40",
    "returned_payment_fee": "Up to $40",
    "cash_advance_fee": "$10 or 5%",
    "promotional_apr": null,
    "promo_apr_months": null,
    "grace_period_days": 21
  },
  "last_updated": "2026-05-15T00:00:00"
}
```

**Tips:**
- Use when the user asks about interest rates, carrying a balance, or penalty fees
- `last_updated` indicates when terms were last extracted — flag if stale
- Terms data may not be available for every card; check the `message` field if `terms` is absent

## get_card_changes

Get an audit log of recent changes to a card's data.

**Parameters:**
- `card_name` (string, required): Card name (fuzzy matching applied)
- `since_days` (int, optional, default 90): How far back to look
- `field` (string, optional): Filter to a specific field, e.g. "annual_fee", "rewards_multipliers_json", "key_benefits", "sign_on_bonus"

**Returns:**
```json
{
  "success": true,
  "card": "American Express Gold Card",
  "since_days": 90,
  "change_count": 2,
  "changes": [
    {
      "field": "annual_fee",
      "old_value": "250",
      "new_value": "325",
      "detected_at": "2026-04-01",
      "source": "bankrate"
    },
    {
      "field": "key_benefits",
      "old_value": "...",
      "new_value": "...",
      "detected_at": "2026-04-01",
      "source": "issuer"
    }
  ]
}
```

**Tips:**
- Use when a user asks "has anything changed on my card?" or "did the fee go up?"
- If `change_count` is 0, tell the user no changes were detected in the period
- Combine with `check_card_renewal` when a card's value may have shifted

## get_program_trends

Get CPP and transfer partner ratio trends for a points program.

**Parameters:**
- `program_key` (string, required): Program identifier — e.g. "chase_ur", "amex_mr", "marriott_bonvoy", "hilton_honors", "delta_skymiles"
- `since_days` (int, optional, default 180): How far back to look

**Returns:**
```json
{
  "success": true,
  "program": "Chase Ultimate Rewards",
  "program_key": "chase_ur",
  "current_portal_cpp": 1.5,
  "since_days": 180,
  "change_count": 1,
  "changes": [
    {
      "field": "portal_cpp",
      "old_value": "1.5",
      "new_value": "1.5",
      "detected_at": "2026-03-15",
      "source": "manual"
    }
  ]
}
```

**Tips:**
- Use when a user asks "has Chase UR lost value?" or "are Amex MR transfer rates changing?"
- `current_portal_cpp` is the current cents-per-point value through the program's travel portal
- Combine with `get_card_changes` for a full "health check" on a card and its program
- If `change_count` is 0, the program valuation has been stable
