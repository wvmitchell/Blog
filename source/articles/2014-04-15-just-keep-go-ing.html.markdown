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

Hmm, 45 seconds to get data from 50 websites, not ideal. Now lets see how we can
achieve somthing similar in Go:

  ```go
    package main

    import (
      "net/http"
      "fmt"
      "time"
    )

    var successes = 0

    func fetchSite(site string) {
      response, _ := http.Get(site)
      if response.StatusCode == 200 {
        successes += 1
      }
    }

    func main() {

      // again, just two sites 25 times
      websites := []string{"http://www.google.com", "https://www.facebook.com"}

      for i := 0; i < 25; i++ {
        for _, site := range websites {
          go fetchSite(site)
        }
      }

      // Slow down for a moment so the goroutines can finish
      time.Sleep(time.Second)

      // Print the number of sites which returned a status code 200
      fmt.Println("Successful calls:", successes)

    }
  ```
This isn't exactly the same, but it achieves a similar result. 25 times were
Getting a response from facebook and google, and each time the response status
code is a 200, that is to say ok, we're recording this. At the end of the
program, we're printing out the number of ok responses. So, speed test?

  ```bash
    $ time go run fetcher.go
    Successful calls: 50

    real    0m1.906s
    user    0m0.926s
    sys     0m0.140s
  ```
So, what's this mean? This Go code compiled, fetched data from 50 websites, and
returned the results of those queries all in less time than it takes to fire up
the ruby interpreter. How?

The critical piece in the Go code above is the line:

  ```go
    go fetchSite(site)
  ```
This line starts up a <a
href="http://golang.org/doc/effective_go.html#goroutines" target="blank">
goroutine</a>, a lightweight thread of execution that runs
autonomously from the main program. This allows the fetchSite function to get
called and run in the background, without blocking the for loop which it's
inside of from looping on.

Inside the context of the loop, we start up 50 goroutines in only the amount of
time it takes to run through the loop itself. These goroutines are then
executing concurrently, and thus only take about as long to finish as calling
the fetchSite function once. Pretty cool huh?

Now, currently, the method we're using for our goroutines isn't ideal. Right now
we're just starting them up, then slowing down our program for a second to allow
the goroutines to finish executing. A better way of doing this would be to use
<a href="http://golang.org/doc/effective_go.html#channels" target="blank">
channels</a>.

Channels are a way of communicating between goroutines. In this
case our fetchSites method is launching 50 goroutines and pushing the results of
this into a channel. In the main function, were listening on this channel for
messages to recieve, using the select statement. If we've listened on the
channel for more than a second, we time out.

  ```go
    package main

    import (
      "net/http"
      "fmt"
      "time"
    )

    func fetchSites() <-chan int {
      c := make(chan int)

      websites := []string{"http://www.google.com", "https://www.facebook.com"}

      for i := 0; i < 25; i++ {
        for _, site := range websites {
          // now launch goroutines inside the function
          go func() {
            response, _ := http.Get(site)
            c <- response.StatusCode
          }()
        }
      }
      return c
    }

    func main() {

      c := fetchSites()
      var successes = 0

      for {
        select {
        case code := <-c:
          if code == 200 {
            successes += 1
          }
        case <-time.After(time.Second):
          fmt.Println("Successful calls:", successes)
          return
        }
      }
    }
  ```

We see running this code that it executes in roughly the same time, however it's
not too difficult to imagine the benefits of operating this way.

  ```bash
    $ time go run fetcher.go
    Successful calls: 50

    real    0m1.876s
    user    0m1.008s
    sys     0m0.137s
  ```
By creating functions which return channels, such as our fetchSites function, we
enable the main function to listen for activity on these various channels, and
take action accordingly.

So far, I'm loving this language. Learning anything new is always a bit tricky,
and I've had more than a few moments of wanting to put my fist through my
computer out of frustration. Unlike Ruby, Go is very opinionated on how things
should be done. While this is great once you wrap your head around these
concepts, it can make finding solutions very difficult at first. I've still got
a ways to make my 100 day goal, so I'm sure there will be lots of new nuggets to
share in the future.

In the mean time, if you have any experiences with Go you'd like to share,
please feel free to reach out. Thanks!

-Will
