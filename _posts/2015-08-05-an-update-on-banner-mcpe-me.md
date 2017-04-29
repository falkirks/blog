---
title: An update on banner.mcpe.me
layout: post
permalink: an-update-on-banner-mcpe-me
published: true
---
I have been involved with PocketMine Banners for (almost) a year now. For those who don't know, PocketMine Banners allows you to generate images which indicate your MCPE server status. The service is used by many MCPE server communities. It has been an great learning experience for me. We (me and Leon Chang) migrated over to a new backend a few months back and although we hit some bumps, we are still generating thousands of banners every day. I would like to inform everyone of the current direction of the service and how we will ensure its continued operation.

The service backend is written in PHP and the frontend is written in JavaScript (some JQuery). Our server is Apache and runs on a single heroku dyno (somewhere in the US). We host our DNS through CloudFlare, this allows us to cache banners in case of an emergency. 

The key problem is speed, we are having to render off every image in succession on a single dyno. This is pretty crappy when you think about it. We need money to scale and our current buisness model doesn't help us.

We have been experimenting with ads over the past few weeks. These ads will **never** in a million years, appear in banners. These ads will hopefully provide us with revenue that will allow us to scale onto more powerful hardware. 

I am also excited to devote some time to write the third iteration of our backend in Node.js. This is long term goal and might take several months to enter testing. I am still deciding the particulars of what the backend will run on, but you can expect a large speed increase. 

Also on the TODO list, I am planning to add support for servers without query enabled in the new backend.

I love permissive licensing and the open source community and I hope that as I work on the third, we will be able to share our code on GitHub. The existing banner related code on GitHub is outdated and the current backend isn't modular enough to extract an open source version of the software.

It is of paramount importance to us that everything remain backwards compatible. Our current service is compatible with the very first banners we ever generated. We offer a dozens of different URL formats and we will continue to support these through to the crack of doom.

Please report any bugs and send feature requests to <a href="mailto:banner@mcpe.me">banner@mcpe.me</a>. 

You can make a banner of your own at http://banner.mcpe.me.
