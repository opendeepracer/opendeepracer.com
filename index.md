# Open Deepracer

The AWS Deepracer is an autonomous driving model car.  It is 1:18 scale model with a fixed camera, Ackermann steering and a throttle all controlled by an Intel Atom board. It uses reinforcement learning in a simulator to learn to drive. 

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

