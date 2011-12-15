---
layout: post
title: Riding the Unicorn
---

<h4>{{ page.title }} - {{ page.date | date_to_long_string}}</h4>

<hr>

<p />
Unicorn is a great rack server.  Looking at thin and passenger they sure seem appealing but there is something about unicorn that 
feels graceful.  I'm not sure if it's the hot restarts, the socket connection or the forked workers that give me that warm and 
fuzzy feeling inside.  Sometimes I still get sad when I have to kill a unicorn process.  It's nice to sit back and watch the master 
and the small herd of workers in the server farm grazing.
<p />
So what do you do to keep these wild stallions in line?
<p />

<h4>Configuration</h4>
Let's start with the unicorn configuration file.  Github has <a href="https://github.com/blog/517-unicorn">a great article</a> on 
the ins and outs, but let's talk about some of the configuration options in <a href="https://gist.github.com/206253">their config.</a>

<p />

<strong>worker_processes (rails_env == 'production' ? 16 : 4)</strong>

<p />

worker_processes are the amount of forked workers that will run on each unicorn instance.  This can make or break your unicorn install. 
16 will actually spawn 17 ruby instances, 1 being the master process.  The master process doesn't actually do any work, it's responsible 
for managing the workers and is a base for the forked processes.  Most articles seem to say match this loosely to the number of cores in 
your system, this is generally a safe bet.  In the day and age of the cloud and virtualization this isn't always correct.  Every app is 
different but you wan't to match up values against your cpu utilization and memory usage.

<p />

cpu

<p />

If you do not have any monitoring tools the easiest way to measure current cpu usage is through the top command.  Load average is a 
pretty standard way to view cpu utilization but ensure you are measuring against the <a href="http://en.wikipedia.org/wiki/Load_(computing)"> 
correct values for your system</a>.  You can also measure with the percentage statistics in top.  You want to look at %idle(id) 
and <a href="http://blog.thinrhino.net.in/cpu-steal-time">%steal(st).</a>  Steal is a new value in virtualized environments and in simple 
terms its cpu your virtualized server had to wait for to be allocated to an actual physical processor.  The timing on this is based on
your hypervisor.

<p />

memory

<p />

Each app has a different memory footprint.  You want to align your memory usage to the actual memory allocated to your machine.  In most
environments your machine is configured with physical memory and swap.  You need to look at your OS footprint and look at how much actual 
memory you have available on your server. Do not count swap in this calculation.  You never want to start swapping to disk, this is slow
and will kill the performance of your server.  Using monitoring which is in detail below we can keep our unicorns at a maximum memory limit.

<p />

<strong>preload_app true</strong>

<p />

This is a very important setting of unicorn.  With this set to true the master process will boot and the workers will be forked when the 
master process has booted.  With this in place your master process may take a few seconds to boot but any subsequent workers will be nearly 
instant to bring online.  Through signals unicorn workers can be added and removed on demand and with preloading the app this is near instant.

<p />

<strong>timeout 30</strong>

<p />

This setting is related to long running processes.  The default value is 60, which in most environments should be ok but decreasing this to 30 
helps keep your long running processes from tying up your workers.  30 seconds is a long time for a normal request so we want to kill these 
before they extend any longer.  If you have intentional long running processes you'll need to fine tune this variable.

<p />

<strong>listen /path/to/socket, :backlog => 2048</strong>

<p />

This setting is how your upstream connections are setup.  You can listen on unix sockets or tcp/ip.  Sockets are great as the requests drop in 
to a queue and workers just grab what they can when they are free.  This ensures a single worker is never overloaded with requests while other
workers sit idle.  Listening on tcp/ip is great if you intend to have unicorn itself shuffle processes across boxes but typically you would 
handle this on your load balancer.  The backlog setting goes hand in hand with the queue.  When the backlog value is reached in the queue 
unicorn will stop handling requests.  This is a valuable setting as when a server is being overloaded with requests it will not continue to 
pile up requests in the queue taxing the server to death.  Setting this to a lower value can help in your load balancing if you've 
configured proper failover.  1024 is the default value.

<p />

<strong>after_fork, before_fork</strong>

<p />

These are the tasks that run before and after the fork of the worker processes from the master.  These will be specific to your environment.

<p />

<h5>Other Settings</h5>
Some other settings that you may want to consider

<p />

<strong>pid /path/to/pid/file.pid</strong>

<p />

pid file for unicorn. This can be important for monitoring the unicorn process.

<p />

<strong>working_directory</strong>
working directory for newly spawned instances.  Should be set to your app root.

<p />

<strong>stderr_path and stdout_path</strong>

<p />

paths to log files.  For troubleshooting and or log watching it's good to have unicorn in seperate log files.

<p />

Don't just follow this post.  Take a look at the unicorn <a href="http://unicorn.bogomips.org/TUNING.html">TUNING</a> and 
<a href="http://unicorn.bogomips.org/DESIGN.html">DESIGN</a> documents from their site.  More technical info on the 
<a href="http://unicorn.bogomips.org/Unicorn/Configurator.html">Configurator</a> page.

<p />

<h4>Monitoring</h4>
Now that you've got unicorn configured let's look at monitoring.  Monitoring unicorn for me is about 2 things, alerting and service management.  
I want to be alerted when there is an issue, but only when I need to take action and I want the service to fix itself or run maintenance when 
needed.

<p />

<h5>Alerting</h5>
For alerting, I use nagios.  Nagios is a solid and reliable monitoring tool and we can use 2 checks for unicorn.  First we can make sure
a unicorn is running.  In NRPE we can specify our check_unicorn command.

<script src="https://gist.github.com/1480080.js"> </script>

Second we can check the current unicorn backlog

<script src="https://gist.github.com/1480086.js"> </script>

The check_raindrops code here is

<script src="https://gist.github.com/1480096.js"> </script>

With these settings an alert will go out if unicorn isn't running or if a box is overloaded, both things I wan't to alerted about.  Even 
with god ensuring unicorn is running, if it can't start we have a problem.

<p />

<h5>Service Management</h5>
God is a process monitoring framework that uses ruby DSL.  This is appealing if you are already running rails and other ruby DSL such as chef.

<p />
For god the <a href="https://github.com/blog/519-unicorn-god">github setup</a> is pretty awesome.  With this it ensures unicorn is always running, 
gives you easy start/stop tasks for the process, and monitors memory and cpu utilization restarting where necessary.  This is the perfect way 
to setup maximum memory use for unicorn allowing you to not overstep your bounds and hit swap.

<p />

<h4>Logging</h4>
Finally, what do we do about our production.log rails log file or our error log file?  These are going to get big quick so we need to rotate them.

<p />
Unicorn actually provides a <a href="http://unicorn.bogomips.org/examples/logrotate.conf">great example file</a> for how to logrotate.  I use a 
slight modication to this to include the unicorn error log and I drop this into /etc/logrotate.d/

<script src="https://gist.github.com/1480137.js"> </script>

<p />

