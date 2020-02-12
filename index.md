# We are Open Deepracer...

Lorem ipso etc

## Blog

Last 10 posts

<ul>
{% for post in site.posts %}
	{% if forloop.index <= 10 %}
	    <li>
	        <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }} ({{ post.date | date: "%Y-%m-%d"}})</a>
	    </li>
	{% endif %}
{% endfor %}
</ul>

[All posts](/blog)

