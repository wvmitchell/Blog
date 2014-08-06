---
title: Elasticsearch and Chewy
date: 2014-08-06 01:10 UTC
tags: Ruby, Rails
---

Whew...well, it's been far longer than I would have liked since my last
post, but I managed to get myself one of these full time job things, and
as such watched most of my free coding time vanish. Not to worry though,
the job is going great, and I'm learning new techniques at a break neck
pace. To that end, I thought it was about time I shared some of my new
found knowledge.

A common problem in growing data sets is searching and sorting. Once the
amount of records you're grows past just a few hundred, scrolling
through all of your data is just not a viable option. There are any
number of ways you may choose to tackle this problem, however the
solution I'm focusing on today is Elasticsearch, paired with the Chewy
gem.

Elasticsearch is a wizbangpop which sits alongside your application
holding a text representation of all your data. When data is updated or
created from your application, Elasticsearch can update it's own dataset
to match whats currently in your database. The benefit you get from this
is a highly customizable full text search and retrieval, which is
considerably faster than retrieving data objects through standard
ActiveRecord querying.

Lets say we had a database of books, and we wanted to find every book
with the word 'potter' somewhere within the title, or the description.
Using ActiveRecord by itself, this becomes clunky, and not very
maintainable. What if on my next search I only want exact matches? Or I
would like find records where 'potter' is contained within the scope of
a larger word, such as 'potters'? This is where Elasticsearch really
shines.

I should note that Elasticsearch is equipped to index a wide range of
databases, and using ActiveRecord along with some SQL style database is
not a requirement. However, it is how this example will proceed.
