---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
<h2 class="col-header dark-orange">Posts</h2>
{% for post in site.posts %}
  {% if post.title != "Welcome to Jekyll!" %}
  <div class="post-preview">
    {% if post.image or page.image_alt %}
    <img class="post-preview__left" src="{{ post.image }}" alt="{{ page.image_alt }}">
    {% endif %}
    <span class="post-preview__right">
      <a class="preview-title" href="{{ post.url }}">{{ post.title }}</a>
      <span class="date-tag">
        {{ post.date | date_to_rfc822 }} | tags: <em>{{ post.tags | join: "</em> - <em>" }}</em>
      </span>
    </span>
    <div class="post-preview__excerpt"><q>
    ...
    {% if post.content contains "<!--exstart-->" %}
      {{ post.content | split: "<!--exend-->" | first | split: "<!--exstart-->" | last }}
      {% comment %}
      {% assign pex =  post.content | split: "<!--exend-->" | first | split: "<!--exstart-->" | last %}
      {% for i in (0..pex.size) %}
        {{ pex | slice: i }}
      {% endfor %}
      {% endcomment %}
    {% else %}
      {{ post.excerpt }}
    {% endif %}
    ...
    </q></div>
  </div>
  {% endif %}
{% endfor %}
