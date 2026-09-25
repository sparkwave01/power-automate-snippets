# 001 · Compose

**Compose** returns whatever you put into it, unchanged. Two practical uses:

## 1. Debug an expression
Add a Compose, paste the expression you want to check, run the flow, and read the
**Outputs** in the run history.

```
formatDateTime(addDays(utcNow(), 7), 'dddd, dd MMM yyyy')
```

## 2. A "wait here" point after parallel branches
When a flow does several independent jobs (loops, a Scope, a Condition…), run them as
parallel branches and add one Compose below all of them, e.g. named `Compose as Node`.
Every step after it runs only when all branches have finished. After it, the flow can
split into new parallel branches again.

```
Trigger
 ├─ Branch A: Scope + Condition
 ├─ Branch B: Apply to each (list 1)
 └─ Branch C: Apply to each (list 2)
        ↓
Compose "Compose as Node"
        ↓
 ├─ Branch D: Filter array → Apply to each
 └─ Branch E: Filter array → Apply to each
```

The Compose doesn't need real data. Put a short note in **Inputs**, such as `All branches done`.

### Gotcha: one failed branch skips everything after it
By default the joining Compose runs only if **every** branch succeeded. If one branch fails,
the Compose is skipped, and so is every step after it.

To continue anyway: Compose → **Settings → Run after** → expand each branch and also tick
**Has failed** (and **Is skipped** / **Has timed out** if needed). The designer then shows that
branch's connector as a dashed line. Add your own error handling after it, so failures are
still noticed.

📎 LinkedIn post: _link coming soon_
