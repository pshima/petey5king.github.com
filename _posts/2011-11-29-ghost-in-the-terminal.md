---
layout: default
title: Ghost in the Terminal
---




{{ page.title }}

OOOOoooooooOOoOooooOOOhhhhh.  That's the sound of the ghost in os x lion that continues to reopen my terminal states, even if I've closed them properly.  No I don't want to reconnect to all my existing ssh sessions when I've closed them all.  No I don't want to go back into my dev directory this time!  Please no, no, no HELPPPP HELPPPPPL HEEEHHHLLPPSPE.  Simple trick seems to have done this for me.


Ghostbuster


- Quit Terminal
- Go to Finder
 * Option - Go Menu - Library
 * Open "Saved Application State"
 * Open com.apple.Terminal.savedState
 * Delete all files
 * CMD I
 * Check the "Locked" checkbox
 * Open Terminal
 * Hooray. No more ghosts.


