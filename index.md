---
title: Esercizi sulla Lingua di Azure AI
permalink: index.html
layout: home
---

# Esercizi sulla Lingua di Azure AI

Gli esercizi seguenti sono progettati per supportare i moduli in Microsoft Learn per lo [sviluppo di soluzioni in linguaggio naturale](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).


{% assign labs = site.pages | where_exp:"page", "page.url contiene '/Instructions/Exercises'" %} {% per l'attività nei lab %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
