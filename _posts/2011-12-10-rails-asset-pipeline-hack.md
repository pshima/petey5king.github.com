---
layout: post
title: Rails Asset Pipeline Hack
---

<h4>{{ page.title }} - {{ page.date | date_to_long_string}}</h4>

<hr>

<p />

One great feature about unicorn is the hot restart.  The <a href="http://www.github.com">GitHub blog</a>
has a <a href="https://github.com/blog/517-unicorn">great post</a> about unicorn.  The gist of the hot 
restart is you send the existing unicorn master a USR2 signal, it boots up a new master and spawns new 
workers.  The new workers kill the old master.  This is a great setup as it means literally no downtime
for your site.  If you like to deploy as much as I do, it's hard to think about other options.

<p />

So what do you do about your dynamically generated assets?  Well, you move to the rails 3.1 asset 
pipeline.  Until then a couple tricks will keep your site loading up assets during hot restarts.  If
you are using capistrano to deploy and nginx as your front end this might help.

<p />

<h4>Capistrano</h4>

First set this up in your deploy.rb or if using multistage you can drop it in the staged .rb file.
The below sets up an oldassets directory which will now contain the assets from the last release and 
runs it after it updates the new release dir with the new code.
<script src="https://gist.github.com/1458915.js"> </script>

<p />

<h4>Nginx</h4>

As for nginx all you need to do is drop a try_files in to your config for the assets paths.  If you are 
already doing caching for your assets then you may already have the location block in place.  With this 
setup it will check the uri, if it doesn't exist it checks the /oldassets dir, and if neither drops a 
404.  This means when rails generates the assets, nginx will start serving the new ones and until then 
you've got assets from the old release.
<script src="https://gist.github.com/1458969.js"> </script>

<p />

And just like that deploy without any missing css or js.

<p />















