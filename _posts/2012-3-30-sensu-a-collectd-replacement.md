---
layout: post
title: Sensu - As A Collectd Replacement
---

Sensu - As A Collectd Replacement
============

Preface
------------
A month or so ago I finally had some time to start looking in to graphing metrics for our infrastructure.  When we moved off [Engine Yard](http://www.engineyard.com/) we also lost the graphing that went along with it and we needed something to review historical information outside of our app, and outside of what [New Relic](http://newrelic.com/) offered.  There were 2 high level goals for graphing, get at least what we had on [Engine Yard](http://www.engineyard.com/) and provide easy in app graphing for the dev team.

Looking around a lot of people are running [graphite](http://graphite.wikidot.com/), [collectd](http://collectd.org/), [statsd](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/), [cacti](http://www.cacti.net/), [zabbix](http://www.zabbix.com/), [zenoss](http://www.zenoss.com/), [monit](http://mmonit.com/monit/) or a [number of other tools](http://en.wikipedia.org/wiki/Comparison_of_network_monitoring_systems).  I wanted something modern that would work with what we had.  [Collectd](http://collectd.org/) was what was running on Engine Yard so naturally started there.  

Collectd
------------
[Collectd](http://collectd.org/) works pretty good and as it's written in C is fast, solid and lightweight.  A [cookbook or two later](https://github.com/AtariTech/cookbooks/tree/master/collectd) I was up and running with some graphs and a web interface.  A few plugin additions and was off to a good start.  Had a few missing images but wasn't too worried about it.  Started looking into graphite and a collectd plugin to push the stats, later planning on eliminating the collectd web interface.  One big need I had was getting metrics from [redis](http://redis.io/).  This looked easy enough, just rock the redis plugin and done.  Not so much.  Looks like the default collectd package was 4.6 and 4.9 was a minimum for one of the redis plugins.  So then it was either write a cookbook to install from source or just manually install an upgraded package.  After hacking up the cookbook got 4.9 installed and the python plugin going after a bit of headache.  All well and good but what do I do about graphing needs without a plugin?  Writing python collectd plugins for everything wasn't high on my list, I'd rather write a ruby plugin with outputs into statsd, which was the plan.

Bucky
------------
[Bucky](https://github.com/cloudant/bucky) was pretty awesome for when I had it running.  Bucky is a simple collectd and statsd adapter for graphite.  Run a small python script and now you've got a statsd server and a place to drop your collectd data and output it all to graphite.

Putting it all together
------------
This seemed like a pretty decent setup, not overly complex but a few moving pieces and mixes of languages.  I still needed something to background the new statsd scripts I was going to run.  Not too difficult to drop in a few cronjobs but kind of messy.  If I have 50 or a 100 of these I didn't want to have to a cronjob for each.  Then I ran back into sensu.

Sensu
------------
[Sensu](https://github.com/sonian/sensu) is a monitoring framework that aims to be simple, malleable, and scalable.  The title of this post isn't accurate.  [Sensu](https://github.com/sonian/sensu) is much more than a collectd replacement, it's a router for collecting and manipulating information with ruby DSL and replacing collectd (or nagios altogether) is just a start.  The possibilities are endless with big leaps in small chunks of time.  Now I'm collecting data from public apis or internal infrastructure, and outputting to hipchat or geckoboard or graphite.

I first saw sensu at the chef hackaday in Seattle.  This looked pretty awesome and I made a mental note of it.  Months later I finally got around to checking this out.  The quick sensu setup was with a [vagrant up file](https://github.com/sonian/sensu/blob/master/dist/chef/Vagrantfile).  Wow cool! I had wanted to try out [Vagrant](http://vagrantup.com/) as well so a couple quick installs and my first sensu box was up and running.  What I really liked about sensu was the flexibility and simplicity of the system.  With ruby scripts this meant a single language and a single system for any metric I needed.  That's powerful.  

So a few plugings later and I tossed out collectd, and bucky and I was on a righteous path to graphing freedom.  Writing the plugins were easy.  Even writing plugins for software I hadn't used was easy.

It's no joke - checkout the redis plugin.  The code outside of the plugin framework is 5 lines.  The ease and simplicity of this kills what I saw with collectd.  Collectd is a great piece of software but for my needs I needed something more flexible.

<script src="https://gist.github.com/2259915.js"> </script>

Setup
------------
* Single Sensu Server
* Chef Cookbook - [https://github.com/sonian/sensu/tree/master/dist/chef](https://github.com/sonian/sensu/tree/master/dist/chef)
* Community Plugins - [https://github.com/sonian/sensu-community-plugins](https://github.com/sonian/sensu-community-plugins)

Configuration:

<script src="https://gist.github.com/2260113.js"> </script>














