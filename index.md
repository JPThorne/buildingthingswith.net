---
description: "A personal blog of developer notes on .NET, code generation, TDD, Docker, Kubernetes and ESP32 microcontrollers."
---

## Hello World

### Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
