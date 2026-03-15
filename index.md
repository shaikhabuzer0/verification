---
layout: default
title: Home
---

# Design Verification Blog

SystemVerilog • UVM • Computer Architecture • Chip Design

---

{% for post in site.posts %}

<div class="post-card">

<h2>
<a href="{{ post.url | relative_url }}">
{{ post.title }}
</a>
</h2>

<p>
{{ post.excerpt }}
</p>

<p class="date">
{{ post.date | date: "%B %d, %Y" }}
</p>

</div>

{% endfor %}
