---
title: Blogging with Hugo and GitHub Pages
date: 2022-08-17 00:17:00
keywords:
  - Hugo
  - GitHub Pages
---

## Decisions, Decisions
When I went to create this blog, I quickly hit decision paralysis. Should I use a CMS? Which one? Strapi? Ghost? Sanity? What about a statically generated site? Hugo, Jekyll, Gatsby? What about Next.js? What about classic and reliable WordPress? Where should I host?

After hours of research, I finally decided on Next.js and Strapi. Over-engineered for the purpose of this blog, but I thought learning this stack would be interesting. That is… until I looked at hosting cost. Hosting through Digital Ocean with the Strapi-recommended CPU and memory was going to be $50/mo, so I scrapped everything and went with basic Hugo and GitHub Pages. Hugo because I found a theme that I liked, GitHub Pages because it’s free.

## Hugo
Hugo is a static site framework, meaning the entirety of my blog will be written in markdown and checked into [GitHub](https://github.com/seastcott/seastcott.github.io). I found the [Cactus theme](https://www.takuzen.me/hugo-theme-cactus/) which is exactly what I was looking for. Simple, minimal. No bells, no whistles. My only gripe was the literal cactus logo and the light green color, so I changed those.

## GitHub Pages
I had a hiccup with linking my custom domain _steve_._codes_ to GitHub Pages _seastcott.github_._io_. In my domain’s DNS settings, I added a CNAME record with host _www.steve_._codes_ and value _seastcott.github_._io_. This did not work as expected. I fundamentally misunderstood the meaning of a host and how it was different than a domain.

Host and domain are usually used together in the form _host.domain_._com_, i.e. _steve_._codes_ is the domain and _www_ is the host. _www.steve_._codes_ is technically a valid host, but that would leave us with _www.steve_._codes.steve.codes_. With _www.steve_._codes_ set as the CNAME record host, I spent hours wondering why navigating to _www.steve_._codes_ was failing with a DNS error. I threw in the towel thinking it was caching issue that would eventually resolve on its own (nope). I ran this problem by a friend the next day and fortunately he set me straight.

[insert image 1]
[insert image 2]

And we’re live! Though, I started wondering, how scalable is GitHub Pages? It is free after all. Sure enough, I found a site size limit of 1 GB and a bandwidth limit of 100 GB per month. I’ll  definitely never exceed these, but it’s worth exploring what my next-best option would be. Would I need to pay for a dedicated cloud instance? I’ll explore this more in my next post.
