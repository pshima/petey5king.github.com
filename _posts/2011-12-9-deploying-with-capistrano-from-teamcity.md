---
layout: default
title: Deploying with Capistrano from Team City
---

<h4>{{ page.title }} - {{ page.date | date_to_long_string}}</h4>

<hr>

<p>
  We use Team City for Continuous Integration.  For a rails app its definitely a good setup and for free hard to beat.  I'd never used it before but 
  given the quality of Ruby Mine it certainly seems to hold up to that standard.  I've been reading a lot about Jenkins in the past few months and 
  set that up for a project but it seemed to be missing a few options I was used to in Team City.  I couldn't seem to get the options straight for 
  bundler or gemsets without a fair amount of custom changes.
</p>
<p>
  Using the Engine Yard gem and deployment, no issues, but switching over to capistrano deploys, a compounded problem for automatic deploys to staging.
</p>
<p>
  The first step is to get the deployment working command line.  SSH in to your CI server and drop in to the build agents work dir and run the cap.
</p>
<p>
  The catch for the next part is that Team City doesn't load an environment when running a build.  This means no environment variables.  Normally 
  you might not need to worry about this, but when you are using Net:SSH as capistrano does, along with agent forwarding you'll end up with some 
  issues.  Googling around this most people seem to suggest just copying your private key to the remote server.  This means the forward agent is no 
  longer needed but now you have your keys to the castle on your staging server - not a good idea.
</p>
<h5>1. Generate an SSH Key</h5>
<p>
  When prompted save the key to a safe place or just drop it in to ~/.ssh/teamcity and don't specify a passphrase, just hit enter when prompted.
  <div class="highlight">
  <pre>
  $ ssh-keygen -t rsa -C "your_email@youremail.com"
  </pre>
  </div>
</p>
<h5>2. Add the ssh key to your github repo deploy keys</h5>
<p>
  <div class="highlight">
  <pre>
  Git Project Page -> Admin -> Deploy Keys -> Add Another Deploy Key
  </pre>
  </div>
</p>
<h5>3. Setup your ssh config to use the key</h5>
<p>
  <div class="highlight">
  <pre>
  Host example.domain.com
  HostName example.domain.com
  IdentityFile /path/to/ssh/key/teamcity
  User deploy

  Host *
    ForwardAgent yes
  </pre>
  </div>
</p>
<h5>4. Run ssh-agent</h5>
<p>
  <div class="highlight">
  <pre>
  ssh-agent -a /tmp/ssh-teamcity/agent
  </pre>
  </div>
</p>
<h5>5. Configure TeamCity Environment</h5>
<p>
  <div class="highlight">
  <pre>
  Edit your project configuration on step 6 - Properties and Environment Variables.<br>
  Set HOME to your HOME dir<br>
  Set SSH_AGENT_PID to the agent PID<br>
  Set SSH_AUTH_SOCK to /tmp/ssh-teamcity/agent
  </pre>
  </div>
</p>
<h5>6. Configure the build steps</h5>
<p>
  Setup a new command line build step to add the key to the agent
  <div class="highlight">
  <pre>
  Command Executable: /usr/bin/ssh-add
  Command parameters: /path/to/ssh/key/teamcity
  </pre>
  </div>
</p>
<p>
  Setup a new command line build step to deploy
  <div class="highlight">
  <pre>
  Command Executable: cap
  Command parameters: staging deploy
  </pre>
  </div>
</p>



















