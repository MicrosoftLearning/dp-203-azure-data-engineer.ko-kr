---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# Azure 데이터 엔지니어링 연습

다음 연습에서는 [Microsoft Certified: Azure Data Engineer Associate](https://learn.microsoft.com/certifications/azure-data-engineer/) 인증을 지원하는 Microsoft Learn의 교육 모듈을 지원합니다.

이러한 연습을 완료하려면 관리 액세스 권한이 있는 [Microsoft Azure 구독](https://azure.microsoft.com/free)이 필요합니다. 일부 연습에서는 [Microsoft Power BI 테넌트](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi)에 대한 액세스 권한이 필요할 수도 있습니다.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 연습 | ILT에서 이는 다음과 같습니다. |
| --- | --- |
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) | {{ activity.lab.ilt-use }} |
{% endfor %}
