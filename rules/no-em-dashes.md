# Rule: No Em Dashes (or En Dashes)

All generated output must avoid the em dash and the en dash characters entirely. This keeps the
radar's writing plain, scannable, and consistent across the briefing, the dashboard, the Notion
pages, and the war-room addendum.

## Banned characters

- The em dash, U+2014.
- The en dash, U+2013.

Do not use them anywhere in any generated artifact: briefings, dossiers (including text values copied
from sources should be normalized), dashboard text, Notion content, the council addendum, commit
messages, or PR descriptions.

## Use these instead

- For a strong break in a sentence, use a comma, a colon, or split into two sentences.
- For a numeric range, write "5 to 8" or "5-8" with a hyphen, not "5 to 8" with an en dash.
- For a parenthetical aside, use commas or parentheses.

## When copying quotes from the web

When a review or news quote contains an em or en dash, normalize it to a hyphen or restructure the
punctuation. The quote stays faithful in wording and keeps its `source_url`; only the dash glyph is
swapped. This does not violate [no-fabrication.md](no-fabrication.md) because no fact changes.

## Verification

The packaging step greps every generated file for U+2014 and U+2013. Any hit is a build failure.
Hyphens (U+002D) are fine.
