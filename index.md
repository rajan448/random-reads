---
title: Explainer Books
---

# Explainer Books

Short guides for curious readers. Choose a book to read its overview, then follow the chapter links.

{% assign books = site.pages | where: "name", "00-overview.md" | sort_natural: "title" %}

<ul>
{% for book in books %}
  {% assign parts = book.path | split: "/" %}
  {% if parts[0] == "explainer-books" and parts.size == 3 %}
    {% assign book_title = book.title | default: parts[1] | remove: " — Explainer Book" %}
    <li><a href="{{ book.url | relative_url }}">{{ book_title | escape }}</a></li>
  {% endif %}
{% endfor %}
</ul>

