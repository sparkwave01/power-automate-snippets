# 002 · Try / Catch with Scope, Run after and Terminate

Build this scaffold **right after the trigger, before any real logic**. Adding it to a finished flow
later means moving every action into a Scope, which is slow with parallel branches.

```
Trigger
   ↓
Initialize variable(s)        ← must stay at top level, can't go inside a Scope
   ↓
Scope "Try"                   ← all the real work goes in here
   ↓
Scope "Catch"                 ← Run after (on Try): Has failed + Has timed out + Is skipped (untick Is successful)
   ├─ Filter array "Failed actions"
   ├─ Post message / Send email
   └─ Terminate  (Status: Failed)
```

## Run after for Catch
Catch → **Settings → Run after** → Try: untick **Is successful**, tick **Has failed**, **Has timed out**
and **Is skipped**. *Is skipped* covers a failure before Try (e.g. in Initialize variable), which makes
Try skip entirely.

## Which action failed, and why
**Filter array**, renamed to `Failed actions`

- From:
  ```
  result('Try')
  ```
- Condition (advanced mode):
  ```
  @equals(item()?['status'], 'Failed')
  ```

Then, in your message:

| What | Expression |
|---|---|
| Failed action name | `first(body('Failed_actions'))?['name']` |
| Error message | `first(body('Failed_actions'))?['error']?['message']` |

> `result('Try')` only looks at the actions **directly** inside Try. If the failure happened deeper
> (inside a loop or Condition), you may only get the generic
> *"An action failed. No dependent actions succeeded."* Point `result()` at that inner Scope/loop
> if you need the exact message.

## Link to the failed run
```
concat(
  'https://make.powerautomate.com/environments/',
  workflow()?['tags']?['environmentName'],
  '/flows/',
  workflow()?['name'],
  '/runs/',
  workflow()?['run']?['name']
)
```

## Mark the run as failed
Last action in Catch: **Terminate**, Status **Failed**. Without it, a run where Catch handled the
error shows as *Succeeded* in the run history.

## Optional: Finally
A third Scope `Finally` after Catch, with Run after on Catch set to **all four** statuses
(Is successful, Has failed, Is skipped, Has timed out). Put clean-up steps there.
Note: if Catch ends with Terminate, Finally won't run after a failure — place Terminate at the end of
Finally instead (with a Condition), or skip Finally.

📎 LinkedIn post: _link coming soon_
