---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

# Übungen zur Azure-Datentechnik

Die folgenden Übungen unterstützen die Schulungsmodule in Microsoft Learn, die die Zertifizierung [Microsoft Certified: Microsoft Certified: Azure Data Engineer Associate](https://learn.microsoft.com/certifications/azure-data-engineer/) unterstützen.

Sie benötigen ein [Microsoft Azure-Abonnement](https://azure.microsoft.com/free) mit Administratorzugriff für diese Übungen. Bei einigen Übungen benötigen Sie möglicherweise auch Zugriff auf einen [Microsoft Power BI-Mandanten](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Übung | In ILT ist dies ein/e... |
| --- | --- |
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} – {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) | {{ activity.lab.ilt-use }} |
{% endfor %}
