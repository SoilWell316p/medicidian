---
aliases:
tags:
  - origin
status: false
---
## 概要
---
### other linked files
```dataviewjs
dv.list(
dv.pages(`[[${this.currentFilePath}]]`)
	.map(page => page.file.link))
```
