---
title: Just keep Go-ing
date: 2014-04-15 19:10 UTC
tags: Languages, Go, Ruby
---

A quarter of the way to my goal of 100 days and still going strong! As with
anything, some days have been more fruitful than others, but I'm happy to report
I've thus far met the very modest goal of having at least one github commit in
Go per day. Most days have seen many more. So what sort of highlights do I have
to share?

Go is fast. Blazing fast. Blow your hair back fast. Choose your own metaphor,
but any way you slice it, it's impressive. Being a compiled language, Go is not
dependent on any kind of interpreter or runtime virtual machine. Lets take a look at
some speed comparisons.

Say you wanted to get html from 50 different websites for text processing. In
ruby you might do something like this:

  ```ruby
    require 'open-uri'

    # we'll just use two sites 25 times for brevity
    websites = ["http://www.google.com", "https://www.facebook.com"]

    25.times do
      websites.each {|site| open(site)}
    end

  ```

Save this in a file, and run it while tracking the time. On my machine, the
results look like this:

  ```bash
    $ time ruby fetcher.rb

    real    0m45.682s
    user    0m1.002s
    sys     0m0.116s
  ```

Hmm, 45 seconds to get data from 50 websites, not ideal. Check out how this can
be done in Go.
