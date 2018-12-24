---
layout: default
title: Motivation
nav_order: 10
---
# Motivation
My primary motivation in this is to figure out how _server-side-events_
work.  After reading several treatises on the subject and looking at
a few github modules, I was still not sure how to implement what I
had in mind.

Ultimately, after reading and reading, the next best thing was to
sit down and start working up an example.

Once I had a working example, the next best thing was to figure
out how to encapsulate the salient pieces of the working example into
something that I (and perhaps others) could a) improve in isolation, and b) ultimately use in other applications.

This is the result.

<blockquote style="background-color:#eee; padding:0.5em;        
                   border-radius:0.5em">
(Hey -- along the way, I'm getting a bang-up introduction to <em>npm publish</em>,
<em>github projects</em>, <em>jekyll</em>, and publishing to <em>github pages</em>, not to mention <em>docker</em>.  Whee!)
</blockquote>

---

Note that as of this writing, it's still a work in progress.  I keep going back to the docs and references and tuning things
to my new understanding.

Then, too, things don't necessarily work as specified - on Chrome,
for instance, the 'retry_interval' argument is happily ignored. Chrome sends the re-connects pretty much whenever it feels like it.

But that may be a mis-interpretation, too.

I'll keep updating things as I learn.  Also, if anyone cares to chime
in, please do.

rickb

