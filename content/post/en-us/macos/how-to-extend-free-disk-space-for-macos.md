---
title: "How to Extend Free Disk Space for Macos"
date: 2021-01-04T18:33:47+08:00
lastmode: 2021-01-04T18:33:47+08:00
draft: false
tags: [ "macos", "diskutil" ]
categories: ["macos"]
reward: true
mathjax: true
---


## Extend disk with diskutils

Delete partition "Free Space" to extend disk to current macos partition.

## Fixed Error “The new size must be different than the existing size”

```bash
#repair (internal, disk0 - in your case)
diskutil repairdisk disk0

#resize (synthesized, disk1 - in your case with HS)
diskutil apfs resizeContainer disk1 0
```


## Reference
---
  1. [Cannot resize APFS partition - “The new size must be different than the existing size”](https://apple.stackexchange.com/questions/375136/cannot-resize-apfs-partition-the-new-size-must-be-different-than-the-existing)