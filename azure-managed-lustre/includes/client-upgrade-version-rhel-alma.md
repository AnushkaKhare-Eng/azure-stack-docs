---
author: rayfergusonmsft
ms.author: rayferguson
ms.service: azure-stack
ms.topic: include
ms.date: 10/24/2024
ms.reviewer:
ms.lastreviewed:
---

<!-- markdownlint-disable-file MD041 -->

```bash
sudo dnf install amlfs-lustre-client-2.15.5_41_gc010524-$(uname -r | sed -e "s/\.$(uname -p)$//" | sed -re 's/[-_]/\./g')-1
```
