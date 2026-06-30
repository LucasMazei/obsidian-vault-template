---
tags: [ai-generated]
---
# 🏆 Goals Dashboard

```dataview
TABLE without id link(file.name) as "Goal", type, target, current
FROM #goal
WHERE !contains(file.path, "7. Templates")
SORT type ASC
```
