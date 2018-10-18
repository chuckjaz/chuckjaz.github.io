---
layout: page
---
{% include JB/setup %}

<div class="site-header">
  <center class="site-title">{{ site.title }}</center>
  <center class="site-quote">{{ site.tagline }}</center>
</div>

<div class="side-panel">
  <div class="author">
    <div class="about-author">
      <div class="author-name"> {{ site.author.name }} </div>
    </div>
    <div class="author-description">
      <p>Chuck is a software engineer at <a href="https://google.com">Google</a> working
        on <a href="https://www.android.com">Android</a>.</p>

      <p>Prior to Android he worked on on <a href="https://angular.io">Angular</a> and 
      <a href="https://cloud.google.com">cloud</a> tools.</p>

      <p>Prior to working at Google, Chuck was a software engineer at
      <a href="http://microsoft.com">Microsoft</a> working on the XAML UI library for
      all Microsoft platforms.</p>

      <p>Previously he worked on frameworks and tools to support
      <a href="http://xbox.com">XBOX ONE</a> and
      developed Visual Studio features targeted towards the JavaScript
      programmer that shipped in Visual Studio starting with Visual Studio 2012.
      He also worked on XAML tools incorporated into Visual Studio starting in
      Visual Studio 2005.</p>

      <p>Before Microsoft he work at Borland on Delphi </a>.
      where he was Chief Scientist and Co- then Chief Archtect of Delphi
      (now part of <a href="http://www.embarcadero.com/">Embarcadero
      Technologies)</a></p>
    </div>
  </div>
</div>

<div class="main-content">
<!-- {% increment count %} -->
{% for page in site.posts %}
<h1><a href="{{ BASE_PATH }}{{ page.url }}">{{ page.title }}</a></h1>
<div class="row post-full">
  <div class="col-xs-12">
    <div class="date">
      <span>{{ page.date | date_to_long_string }}</span>
    </div>
    <div class="content">
      {{ page.content }}
    </div>

  <div class="comments-link">
    <a href="{{ BASE_PATH }}{{ page.url }}/#disqus_thread">comments</a>
  </div>

  {% unless page.categories == empty %}
  <ul class="tag_box inline">
    <li><i class="glyphicon glyphicon-open"></i></li>
    {% assign categories_list = page.categories %}
    {% include JB/categories_list %}
  </ul>
  {% endunless %}  

  {% unless page.tags == empty %}
  <ul class="tag_box inline">
    <li><i class="glyphicon glyphicon-tags"></i></li>
    {% assign tags_list = page.tags %}
    {% include JB/tags_list %}
  </ul>
  {% endunless %}

  {% if count >= 5 %}
    {% break %}
  {% endif %}
  <!-- {% increment count %} -->
{% endfor %}

<h1>Older posts</h1>
<div class="archive-text">
  See <a href="archive.html">archive</a> for older posts.
</div>
</div>
