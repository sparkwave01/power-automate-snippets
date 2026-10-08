# 003 · Filter array, and item() vs items()

## Filter array instead of a Condition inside a loop

```
Get items
   ↓
Filter array      From: value of Get items      Condition: item()?['Status']  is equal to  Approved
   ↓
Apply to each     over body('Filter_array')     ← only matching rows, no Condition needed
```

Advanced mode version of the same condition:

```
@equals(item()?['Status'], 'Approved')
```

## item() vs items()

| Expression | Means | Where |
|---|---|---|
| `item()` | the element the **current action** is working through | Filter array, Select, and inside Apply to each (innermost loop) |
| `items('Apply_to_each')` | the current element of **that named loop** | anywhere inside that loop, including inside a Filter array or a nested loop |

Spaces in the loop name become underscores: loop `Apply to each Customer` → `items('Apply_to_each_Customer')`.

## Filter array inside a loop (the confusing case)

Loop through customers, and for each customer filter the orders loaded earlier:

```
Get items (Customers)
Get items (Orders)                     ← load once, outside the loop
Apply to each  (value of Customers)
   ├─ Filter array   From: value of Get items (Orders)
   │                 @equals(item()?['CustomerID'], items('Apply_to_each')?['ID'])
   └─ Compose        length(body('Filter_array'))     ← quick check
```

- `item()` → the **order** being checked by the filter
- `items('Apply_to_each')` → the **customer** of the current loop round

If you write `item()` on both sides, the flow doesn't fail. It compares each order with itself and
returns the wrong rows (or none). The Compose count makes this visible: the same number for every
customer, or always 0, means the condition is pointing at the wrong item.

### The designer hides the difference
In basic mode, `items('Apply_to_each')?['ID']` and `item()?['ID']` both show up as the same **ID** token.
Hover over the token to see which expression it really is.

Tested with sample data: for the third customer the correct filter returned 3 orders, the wrong one
returned 0, and every step was green.

> SharePoint **lookup** columns come back as objects. For a lookup column named `Customer`, use
> `item()?['Customer']?['Id']` instead of `item()?['CustomerID']`.

📎 LinkedIn post: _link coming soon_
