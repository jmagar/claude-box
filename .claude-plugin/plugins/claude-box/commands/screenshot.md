---
allowed-tools: Bash, Read
description: Analyze the latest screenshot from /mnt/c/screens
---

## Context

Latest screenshot: !`find /mnt/c/screens -name "*.png" -type f -printf '%T@ %p\n' | sort -n | tail -1 | cut -d' ' -f2-`

## Your task

Analyze the screenshot above and provide a detailed description of what you see, including:
- UI elements and layout
- Text content visible
- Visual design patterns
- Any code or technical content
- Key functionality or features shown
- Suggested improvements or observations
- Context clues about what application or system this is