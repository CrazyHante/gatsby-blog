---
title: windows端口占用
category: Diary
cover: diary.png
---

1. netstat -aon|findstr "8080"
2. tasklist|findstr "9524"
3. taskkill /f /t /im java.exe

