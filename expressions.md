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

📎 LinkedIn post: _link coming soon_

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

