{{ $platformDirArray := split .File.Dir "/" }}

---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
Platform: {{ index $platformDirArray 1 }}
type: "posts"

---

