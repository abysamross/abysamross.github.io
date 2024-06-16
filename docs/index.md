---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
<h2 class="col-header dark-orange">Posts</h2>
{% for post in site.posts %}
  {% if post.title == "Welcome to Jekyll!" %}
    {% continue %}
  {% endif %}
<div class="post-preview">
 <img class="post-preview__left" src="{{ post.image }}" alt="{{ page.image_alt }}">
 <div class="post-preview__right">
   <a class="preview-title" href="{{ post.url }}">{{ post.title }}</a>
   <div class="date-tag">
     <span>{{ post.date | date_to_rfc822 }} | <small>tags: <em>{{ post.tags | join: "</em> - <em>"}}</em></small></span>
   </div>
 </div>
</div>
{% endfor %}
