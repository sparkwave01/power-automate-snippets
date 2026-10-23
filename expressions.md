# Power Automate expressions

Short, copy-paste-ready expressions from my weekly LinkedIn tips. Newest at the bottom.

## 004: Today's date in a file name

`utcNow()` is UTC. In the evening (US time zones) the UTC date is already tomorrow, so convert before formatting.

Wrong after ~8 PM in New York (summer):
```
concat(formatDateTime(utcNow(), 'MM-dd-yyyy'), '_Daily_Report.xlsx')
```

Convert first, then format:
```
concat(formatDateTime(convertTimeZone(utcNow(), 'UTC', 'Eastern Standard Time'), 'MM-dd-yyyy'), '_Daily_Report.xlsx')
```

Shorter, with the format as the 4th argument:
```
convertTimeZone(utcNow(), 'UTC', 'Eastern Standard Time', 'MM-dd-yyyy')
```

With a day offset (convert **before** `addDays()`):
```
concat(formatDateTime(addDays(convertTimeZone(utcNow(), 'UTC', 'Eastern Standard Time'), variables('date_selection')), 'MM-dd-yyyy'), '_Daily_Report.xlsx')
```

Notes:
- Time zone names are Windows names. `Eastern Standard Time` already includes daylight saving (EDT).
  Others: `Central Standard Time`, `Pacific Standard Time`, `GMT Standard Time`, `W. Europe Standard Time`.
- Test any time of day with a fixed UTC timestamp:
  `convertTimeZone('2026-10-14T01:30:00Z', 'UTC', 'Eastern Standard Time', 'MM-dd-yyyy')` → `10-13-2026`

📎 LinkedIn post: https://www.linkedin.com/feed/update/urn:li:share:7518112317729955840/

## 005: Remove duplicates with union()

There's no `distinct()` function. `union()` returns each value once, so pass the same array twice:
```
union(variables('Locations'), variables('Locations'))
```
`["Boston", "Austin", "Boston", "Denver"]` → `["Boston", "Austin", "Denver"]`

Per row inside a **Select**, e.g. a multi-select column with repeated values:
```
union(item()?['Location'], item()?['Location'])
```

Case-insensitive version (lower-case everything first, using a Select in text mode over the array):
```
Select   From: variables('Locations')   Map (text mode): toLower(item())
union(body('Select'), body('Select'))
```

Notes:
- Case-sensitive: `"Boston"` and `"boston"` are different values.
- Objects only count as duplicates when **every** property matches. To dedupe by one field, Select that field first.
- Quick test in a Compose:
  `union(createArray('Boston','Austin','Boston','Denver','boston'), createArray('Boston','Austin','Boston','Denver','boston'))`

📎 LinkedIn post: _link coming soon_

## 007: A short ID in a file name

`Report.docx` → `Report-a1b2c3.docx`, keeping every dot in the name and the extension.

Two Compose actions first:
- `FileName` — the original name, e.g. `triggerOutputs()?['body/{FilenameWithExtension}']`
- `ShortID` — `toLower(substring(guid(), 0, 6))`

Then:
```
concat(
  join(take(split(outputs('FileName'), '.'), sub(length(split(outputs('FileName'), '.')), 1)), '.'),
  '-',
  outputs('ShortID'),
  '.',
  last(split(outputs('FileName'), '.'))
)
```

| Original | Result |
|---|---|
| `Report.docx` | `Report-a1b2c3.docx` |
| `Budget.xlsx` | `Budget-a1b2c3.xlsx` |
| `Report.v2.final.docx` | `Report.v2.final-a1b2c3.docx` |
| `archive.tar.gz` | `archive.tar-a1b2c3.gz` |

Names without an extension (e.g. `README`). The basic version gives `-a1b2c3.README` (tested); this one gives `README-a1b2c3`:
```
if(
  contains(outputs('FileName'), '.'),
  concat(
    join(take(split(outputs('FileName'), '.'), sub(length(split(outputs('FileName'), '.')), 1)), '.'),
    '-', outputs('ShortID'), '.', last(split(outputs('FileName'), '.'))
  ),
  concat(outputs('FileName'), '-', outputs('ShortID'))
)
```

Notes:
- Meant for everyday file names like `Report.docx` or `Budget.xlsx`. Unusual names (e.g. starting with a dot)
  may need extra handling.
- 6 hex characters ≈ 16.7 million combinations: fine for people to tell files apart. For a link between files that
  must never break, store the full `guid()` in a (hidden) column instead.

📎 LinkedIn post: _link coming soon_

