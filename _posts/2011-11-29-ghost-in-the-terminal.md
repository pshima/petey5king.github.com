---
layout: default
title: Ghost in the Terminal
date: November 29, 2011
---

{{ page.title }} - {{ page.date | date_to_long_string}}

<hr>

<p>
  OOOOoooooooOOoOooooOOOhhhhh.  That's the sound of the ghost in os x lion that continues to reopen my terminal states, even if I've closed them properly.  No I don't want to reconnect to all my existing ssh sessions when I've closed them all.  No I don't want to go back into my dev directory this time!  Please no, no, no HELPPPP HELPPPPPL HEEEHHHLLPPSPE.  Simple trick seems to have done this for me.
</p>
<p>
  <strong>Ghostbuster</strong>
</p>
<div class="highlight">
<pre>
 * Quit Terminal
 * Go to Finder
 * Option + Click the Go Menu 
 * Choose Library
 * Open "Saved Application State"
 * Open com.apple.Terminal.savedState
 * Delete all files
 * CMD + I (Get Info)
 * Check the "Locked" checkbox
 * Open Terminal
 * Hooray. No more ghosts.
</pre>
</div>
