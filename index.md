---
title: Instruções online hospedadas
permalink: index.html
layout: home
---

# Microsoft Learn - Exercícios práticos

Os seguintes exercícios práticos foram projetados para dar suporte ao treinamento do [Microsoft Learn](https://docs.microsoft.com/training/).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| |
| --- | --- | 
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} — {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
