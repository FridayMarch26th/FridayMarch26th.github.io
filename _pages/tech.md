---
layout: default
title: Tech notes
description: 
featured_image: /images/social.jpg
---

<section class="blog">
    Test
	<div class="content-wrap blog-wrap">

		{% for note in site.tech %}

		<article class="blog-post">

			<a class="blog-post__link" href="{{ post.url | relative_url }}">

				<div class="blog-post__image">
					<img src="{{ note.featured_image | relative_url }}" alt="{{ note.title }}">
				</div>

				<div class="blog-post__content">
					<div class="blog-post__info">
						<h2 class="blog-post__title">{{ note.title }}</h2>
						<!--<p class="blog-post__subtitle">{{ note.date | date_to_long_string }}</p>-->
					</div>
				</div>

			</a>

		</article>

		{% endfor %}

	</div>

</section>