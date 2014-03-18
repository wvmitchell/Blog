---
title: The Incredible % Operator
date: 2014-03-18 17:32 UTC
tags: Ruby
---

One of my favorite things about programming is how frequently I find myself
discovering some new tool or technique, even within a domain I'm already
familiar with. I spent some time this weekend playing around with the string
array literal syntax, of which I'm already fairly familiar.


  ```ruby
    %w{one two three}

    #=> ["one", "two", "three"]
  ```

After a bit of sleuthing around this weekend though, I discovered this is a much
richer area of the ruby language than I'd previously thought. One interesting
bit is that the delimiters around the literal can be any number of different
characters, just pick whatever suits your fancy.

  ```ruby
    %w{one two three}
    %w|one two three|
    %w$one two three$
    %w[one two three]
    %w^one two three^
    %w(one two three)
    %w&one two three&


    #  All end up as
    #=> ["one", "two", "three"]
  ```

Now, should you really use dollar signs to delimit a string array literal? Probably
not. But as with many things in ruby, it's nice to know you can.

The rabbit hole goes deeper of course. Lets say we wanted to do some string
interpolation with these arrays. Is that even possible? Why yes, it totally is.
All we need to do is use an uppercase W.

  ```ruby
    two = 2
    %W{one #{two} three}        #=> ["one", "2", "three"]
  ```

In reading more about this topic, I discovered that the % operator is the
starting point for this syntax, and the rest is built off it. This is apparently
a Perl inspired way of quoting strings, borrowed and extended by the Ruby
language. Its uses are varied, and all intriguing.

  ```ruby
    # strings
    %(one two three)            #=> "one two three"
    %q(one two three)           #=> "one two three"
    %Q(one #{1+1) three)        #=> "one 2 three"


    # symbols (ruby > 2.0 only)
    %i(one two three)           #=> [:one, :two, :three]
    %I(one #{"t" + "wo"} three) #=> [:one, :two, :three]


    # shell scripts
    %x(ruby -v)                 #=> "ruby 2.0.0p247"
    %x(date)                    #=> "Tue Mar 18 12:13:52 MDT 2014"

    # regex's
    %r(#{"see"+"saw"})          #=> /seesaw/
    %r(seesaw)i                 #=> /seesaw/i
  ```

All of the above examples can be delimited by whatever special characters you
feel best illustrate the technique, although I'd recommend sticking to (), [],
or {} for consistency and readability.

While I'm sure you'll find, as I did, that your current methods for building
strings are just fine, I think the new notation for arrays of symbols is
particularly useful, as it saves a fair bit of typing. Whichever you choose to
make use of, have fun, and happy hacking!
