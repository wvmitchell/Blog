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

Elasticsearch is an opensource RESTful service which sits alongside your application,
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
not a requirement. However, it is how this example will proceed. When in
doubt, go with what you know.

If you're using ActiveRecord to manage the data within your application,
the Chewy gem is a worthwhile addition when using Elasticsearch. Chewy
wraps the Elasticsearch DSL, and makes it easy to query Elasticsearch
and return the found data in a familiar syntax. I would recommend it
over the Elasticsearch-Rails gem, which has much of the same
functionality without the ease of use.

So, perhaps if you've read this far you'd like to know how to actually
use this in your own project? Well, I'm happy to oblige.

Getting the necessary dependencies installed on your machine is pretty
straightforward. If you're on a Mac and have homebrew installed:

  ```bash
    $ brew install elasticsearch
    $ gem install chewy
  ```

If you're not using homebrew, you'll need to download and install 
Elasticsearch manually. Checkout the
<a href="http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/_installing_elasticsearch.html" target='_blank'>installation instructions</a> for more info.

Once installed, you'll need to do a bit of configuration to get
Elasticsearch running locally. Follow the prompts during the install to 
get it working on your machine. Visit 'localhost:9200' to
ensure that Elasticsearch is up and running.

To start using Elasticsearch and Chewy in your Rails project, you'll
need to update your Gemfile.

  ```ruby
    # Gemfile

    gem 'chewy
  ```

Then create a new file, chewy.yml, in the config directory.

  ```yaml
    # config/chewy.yml

    test:
      host: 'localhost:9200'
      prefix: 'test'
    development: 
      host: 'localhost:9200'
  ```

Make sure to bundle after adding to the Gemfile to ensure Chewy is
loaded into your project. Now that we've got everything configured,
we're ready to start defining our indices. Chewy will look inside
app/chewy/ to find the index definitions to load into elasticsearch, so
go ahead and create that. For our index, lets again imaging we have a
Book model in our project that we're managing with ActiveRecord.
Defining they index is super simple.

  ```ruby
    # app/chewy/books_index.rb

    class BooksIndex < Chewy::Index 
      define_type Book do
        field :title, :description, :author
      end
    end
  ```

Chewy is smart enough to know we're talking about the Book class that's
already defined in our project, and conviniently gives some rake tasks
to load in our data.

  ```
    $ rake chewy:update
  ```

Now all of our records are loaded into Elasticsearch! Aweosme, but how
do we get them back? Here is one way you might use your new BooksIndex
in the BooksController:

  ```
    # app/controllers/books_controller.rb

    def index
      if params[:query_string]
        @books = BooksIndex.query(query_string: {query: params[:query_string] } ).load
      else
        @books = BooksIndex.query(match_all: {} ).load
      end
    end
  ```

So, if we have some search term in our params, we'll search our books
based on the search tearm. Otherwise, we'll return all of the records.
Also, the load command on the end of each query will actually load the
data as ActiveRecord objects. This is particularly helpful if you'd like
to display data related to the object that wasn't loaded into your
BooksIndex, such as created\_at or updated\_at.

The final step is telling rails to update the information in
elasticsearch when Book records are created, updated, or destroyed.
Chewy makes this very simple, we just need to add a line to our book
model.

  ```
    # app/models/book.rb

    update_index 'books', 'self', urgent: true
  ```

And that's it! Full text searching never felt so good!

If you're interested in learning more, the Elasticsearch documentation
is your best resource, you can find it <a
href="http://www.elasticsearch.org/" target="_blank">here.</a> The Chewy
gem has a very thorough <a href="https://github.com/toptal/chewy" target="_blank">Readme</a>
, as well as a number of attached gists describing use cases.

I wish you all the best getting Elasticsearch working for you in your
project. The benefits are clear, but the path forward can seem a little
daunting. Hopefully this post gets you there more quickly.
