---
layout: page
title: About Us
cover: /images/tools-rack.jpg
---

# Who are we?

<div id="about-us-authors">
	{% for author_hash in site.data.authors %}
	{% assign author = author_hash[1] %}
	<section class="post-author" itemscope itemtype="http://schema.org/Person">
		<img src="https://github.com/{{ author.github }}.png?size=400" alt="" class="author-avatar">

		<div class="author-info">
			<p><strong itemprop="name">{{ author.displayName }}</strong></p>

			{% if author.punchline %}
			<blockquote class="author-punchline">
				<p>{{ author.punchline }}</p>
			</blockquote>
			{% endif %}
			<p>
				{% if author.twitter %}
				<a href="https://twitter.com/{{ author.twitter }}" class="external-profile" target="_blank" rel="noopener noreferer" aria-label="Twitter profile">
					<i class="fa fa-twitter" aria-hidden="true" title="Twitter profile"></i>
					@{{ author.twitter }}
				</a>
				{% endif %}

				{% if author.github %}
				<a href="https://github.com/{{ author.github }}" class="external-profile" target="_blank" rel="noopener noreferer" aria-label="GitHub profile">
					<i class="fa fa-github" aria-hidden="true" title="GitHub profile"></i>
					{{ author.github }}
				</a>
				{% endif %}

				{% if author.skype %}
				<a href="skype:{{ author.skype }}?userinfo" class="external-profile" target="_blank" rel="noopener noreferer" aria-label="Skype profile">
					<i class="fa fa-skype" aria-hidden="true" title="Skype profile"></i>
					{{ author.skype }}
				</a>
				{% endif %}
			</p>
		</div>
	</section>
	{% endfor %}
</div>
