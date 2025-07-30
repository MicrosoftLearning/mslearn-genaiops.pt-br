---
title: Exercícios de GenAIOps
permalink: index.html
layout: home
---

# Operacionalizar aplicativos de IA generativa

Os exercícios de início rápido a seguir foram projetados para oferecer uma experiência prática de aprendizado, na qual você explorará tarefas comuns necessárias para operacionalizar uma carga de trabalho de IA generativa no Microsoft Azure.

> **Observação**: para concluir os exercícios, você precisará de uma assinatura do Azure na qual tenha permissões e cotas suficientes para provisionar os recursos do Azure e os modelos de IA generativa necessários. Caso ainda não tenha uma conta do Azure, inscreva-se para obter uma [conta do Azure](https://azure.microsoft.com/free). Há uma opção de avaliação gratuita para novos usuários que inclui créditos para os primeiros 30 dias.

## Exercícios de início rápido

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

> **Observação**: embora você possa concluir esses exercícios sozinho, eles foram projetados para complementar os módulos do [Microsoft Learn](https://learn.microsoft.com/training/paths/operationalize-gen-ai-apps/), nos quais você encontrará um aprofundamento em alguns dos conceitos subjacentes nos quais esses exercícios se baseiam.
