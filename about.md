---
layout: page
permalink: /about/index.html
title: Vinay Bhinde
tags: [Vinay, Bhinde, vinaynb]
imagefeature: fourseasons.jpg
chart: true
---
<!-- <figure>
  <img src="{{ site.url }}/images/hossain-faysal.jpg" alt="Hossain Mohammad Faysal">
  <figcaption>Vinay Bhinde</figcaption>
</figure> -->

{% assign total_words = 0 %}
{% assign total_readtime = 0 %}
{% assign featuredcount = 0 %}
{% assign statuscount = 0 %}

{% for post in site.posts %}
    {% assign post_words = post.content | strip_html | number_of_words %}
    {% assign readtime = post_words | append: '.0' | divided_by:200 %}
    {% assign total_words = total_words | plus: post_words %}
    {% assign total_readtime = total_readtime | plus: readtime %}
    {% if post.featured %}
    {% assign featuredcount = featuredcount | plus: 1 %}
    {% endif %}
{% endfor %}


My name is **Vinay Bhinde**, and this is my personal blog. It currently has {{ site.posts | size }} posts in {{ site.categories | size }} categories which combinedly have {{ total_words }} words, which will take an average reader ({{ site.wpm }} WPM) approximately <span class="time">{{ total_readtime }}</span> minutes to read. {% if featuredcount != 0 %}There are <a href="{{ site.url }}/featured">{{ featuredcount }} featured posts</a>, you should definitely check those out.{% endif %} The most recent post is {% for post in site.posts limit:1 %}{% if post.description %}<a href="{{ site.url }}{{ post.url }}" title="{{ post.description }}">"{{ post.title }}"</a>{% else %}<a href="{{ site.url }}{{ post.url }}" title="{{ post.description }}" title="Read more about {{ post.title }}">"{{ post.title }}"</a>{% endif %}{% endfor %} which was published on {% for post in site.posts limit:1 %}{% assign modifiedtime = post.modified | date: "%Y%m%d" %}{% assign posttime = post.date | date: "%Y%m%d" %}<time datetime="{{ post.date | date_to_xmlschema }}" class="post-time">{{ post.date | date: "%d %b %Y" }}</time>{% if post.modified %}{% if modifiedtime != posttime %} and last modified on <time datetime="{{ post.modified | date: "%Y-%m-%d" }}" itemprop="dateModified">{{ post.modified | date: "%d %b %Y" }}</time>{% endif %}{% endif %}{% endfor %}. The last commit was on {{ site.time | date: "%A, %d %b %Y" }} at {{ site.time | date: "%I:%M %p" }} [UTC](http://en.wikipedia.org/wiki/Coordinated_Universal_Time "Temps Universel Coordonné").

I currently live in Ahmedabad, India and work as Web Developer at <a href="http://www.avestatechnologies.com" class="author-website">Avesta Technologies</a>.

From EXTJS to Angular to React to Node.js & Microservices to React Native to PostgresSQL, being in a small team i have been exposed to both the stacks on quite well enough. I have also been heavily involved in setting up DevOps practices and implementing them.

Things that currently interest me are Building/Investigating web apps for performance and working on building an efficient DevOps solutions which enable developers to develop efficiently and release confidently.

I love to work on anything that runs in a web browser.

HTML,<br>
CSS,<br>
JS,<br>
Any of these ring a bell to you ?

Reach out to me over [twitter](https://twitter.com/dilemmist) or <a href="mailto:vinaynb@gmail.com" class="author-website">email</a> and let's talk more of it.