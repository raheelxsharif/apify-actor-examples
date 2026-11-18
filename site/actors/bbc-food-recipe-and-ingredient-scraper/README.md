# BBC Food Recipe & Ingredient Scraper

Turn a BBC Food search or a list of recipe links into structured recipe data. The actor crawls bbc.co.uk/food's search results page by page and reads each recipe's own structured data, so you get the same ingredients, steps, and timings shown on the page itself.

## What this actor does

- **Keyword search** across BBC Food's recipe search, paging through results until `maxItems` is reached
- **Direct URL mode** for a known list of recipe pages
- **Post-fetch filters** by cuisine text match and minimum rating
- **Full recipe body** — ingredients, numbered method steps, timings, servings, and nutrition where BBC Food publishes it
- **Empty fields are omitted** — a recipe without a rating, for example, simply has no `rating` field

## Output per recipe

- `title` — recipe name
- `description` — recipe summary text
- `author` — chef or BBC programme credited with the recipe
- `prepTime`, `cookTime`, `totalTime` — ISO 8601 duration strings (e.g. `PT20M`) as published by BBC Food
- `servings` — recipe yield (e.g. number of servings or portions)
- `ingredients` — list of ingredient lines
- `ingredientCount` — number of ingredient lines
- `instructions` — list of method steps in order
- `stepCount` — number of method steps
- `calories` — calories per serving, when BBC Food publishes a nutrition panel
- `rating` — average community rating
- `ratingCount` — number of ratings behind that average
- `categories` — recipe category tags
- `cuisine` — cuisine type
- `keywords` — up to 10 keyword/tag terms
- `imageUrl` — recipe photo URL
- `datePublished` — original publish date
- `sourceUrl` — the BBC Food recipe page the record came from

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (keyword) or `byUrl` (direct recipe links) |
| `query` | string | – | Keyword used when `mode` is `search`; falls back to BBC Food's "popular" search when left empty |
| `cuisine` | string | – | Keep only recipes whose cuisine or category text contains this value |
| `minRating` | number | – | Keep only recipes rated at or above this value (1.0–5.0) |
| `startUrls` | array | `[]` | Recipe URLs to fetch directly when `mode` is `byUrl` |
| `maxItems` | integer | `20` | Maximum number of recipes to return (1–500) |

### Example: search for a dish

```json
{ "mode": "search", "query": "roast chicken", "maxItems": 20 }
```

### Example: search restricted by cuisine and rating

```json
{ "mode": "search", "query": "curry", "cuisine": "Indian", "minRating": 4, "maxItems": 30 }
```

### Example: fetch specific recipe pages

```json
{ "mode": "byUrl", "startUrls": ["https://www.bbc.co.uk/food/recipes/perfect_roast_chicken_93247"] }
```

## Use cases

- **Recipe app content** — populate a cooking app with structured ingredients and steps instead of raw HTML
- **Meal planning tools** — pull servings, timings, and calories to feed into a planner
- **Content research** — gather a batch of recipes by cuisine for editorial or recipe-roundup work
- **Nutrition tracking** — collect calorie figures across a set of dishes where BBC Food publishes them

## FAQ

**Does this need a login or API key?**
No. bbc.co.uk/food is a public website and the actor reads only its publicly available pages.

**Why do some recipes have no `calories` or `rating`?**
Not every BBC Food recipe page publishes a nutrition panel or carries community ratings. Those fields are left off the record rather than filled with a placeholder.

**What format are `prepTime`, `cookTime`, and `totalTime` in?**
They're kept exactly as BBC Food publishes them — ISO 8601 durations like `PT20M` for 20 minutes or `PT1H30M` for one hour thirty.

**How does the cuisine filter work?**
`cuisine` is matched against both the recipe's cuisine field and its category tags, so a value like `Italian` catches recipes tagged either way.

**Can I scrape a specific recipe directly?**
Yes — set `mode` to `byUrl` and list the exact recipe URLs in `startUrls`; the cuisine and rating filters still apply.

**How many recipes can one run return?**
Up to `maxItems`, capped at 500. In search mode, the actor keeps paging through BBC Food's search results until it reaches that cap or runs out of matches.

**Is this affiliated with the BBC?**
No. This is an independent, third-party actor that reads BBC Food's publicly available recipe pages and is not affiliated with or endorsed by the BBC.
