---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
<style>
.date-tag {
font-style: bold;
}
.post-preview__excerpt {
font-size: 12px;
font-style: italic;
quotes: "\201C" "\201D";
}
</style>

<h2 class="col-header dark-orange">Posts</h2>
{% for post in site.posts %}
  {% if post.title != "Welcome to Jekyll!" %}
<div class="post-preview">
   {% if post.image or page.image_alt %}
 <img class="post-preview__left" src="{{ post.image }}" alt="{{ page.image_alt }}">
   {% endif %}
 <div class="post-preview__right">
   <a class="preview-title" href="{{ post.url }}">{{ post.title }}</a>
   <div class="date-tag"><span><small>
     {{ post.date | date_to_rfc822 }} | tags: <em>{{ post.tags | join: "</em> - <em>"}}</em>
   </small></span></div>
 </div>
 <div class="post-preview__excerpt"><span><q>
   ...
   {% if post.content contains "<!--exstart-->" %}
     {{ post.content | split: "<!--exend-->" | first | split: "<!--exstart-->" | last }}
   {% else %}
     {{ post.excerpt }}
   {% endif %}
   ...
 </q></span></div>
</div>
  {% endif %}
{% endfor %}
