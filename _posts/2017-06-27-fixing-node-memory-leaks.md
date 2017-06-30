---
layout: post
title:  "Fixing Node Memory Leaks"
date:   2017-05-27 13:25:00 -0400
categories: javascript node debugging hacking
---

The morning before leaving for vacation recently, I was on call for team Unicorn Bacon (which has a Node.js application in production) when suddenly my phone lit up like a Christmas tree with alerts. I logged into NewRelic and saw that our service had hung because it had run out of RAM and spiked the CPU. After restarting the process I was able to bring the service back online. There’d been some downtime, but well in advance of when most of our customers started their working day.

Here at Rally, we’ve adopted the habit of blameless post-mortems that we call PERs (post event retrospectives). These are great at getting folks from dev teams, architecture, and ops together to talk about what happened and build a plan towards making sure that we delight our customers and reduce the impact of outages to the business of shipping software. It’s much more effective (and fun) than those finger-pointing outage retros!

During our PER on the outage we identified a few things we’d been missing that would help us get visibility into Node when it was having problems, and allow us to respond before an outage occurred. We formed a working hypothesis that something was leaking memory in our application. After some prioritization and decisions on the team, we first tweaked the memory alert threshold that we had set on the Node hosts to give us more advance warning. We also changed the configuration of Node process to allocate more of the physical RAM on the host for the Node virtual machine.

We also ended up installing [node-heapdump](https://github.com/bnoordhuis/node-heapdump), which gives us visibility into the application and allows us to take heap snapshots and debug them with Chrome’s developer tools. Heapdump adds a user signal that you can pass to the running process to force it to dump heap. We wrote a function in our Node app that runs every 15 seconds and checks the current memory usage. As it approaches 80 percent, it will write a heapsnapshot and then bump the threshold another 100MB, which gives us the escalating snapshot as the heap grows.

Our heapdump.coffee code:

 
````coffee
	
exports.name = 'heapdump'
exports.run = (app)->
   MB = 1024 * 1024
   threshold =1024 * 1024 * 800 # 800 MB
   interval = 1000 * 15 # Every 15 seconds check threshold to dump heap
   increment = MB * 100 # Record heapdump on 100MB increments
   heapdump = require('heapdump')
   moment = require('moment')
   winston = require('winston')
   os = require('os')
   path = require('path')
   setInterval(()->
      usage = process.memoryUsage().rss
      usageInMB = (usage / MB).toFixed(2)
      if usage > threshold
          now = moment().format()
          filename = path.join(os.tmpdir(), "sdpife.#{now}.#{usageInMB}-MB.#{process.pid}.heapsnapshot")
          heapdump.writeSnapshot filename, (err, filename)->
              winston.warn('Wrote heap snapshot', {err, filename, usage, usageInMB})
          threshold += increment
   , interval)
````
 

Two weeks later, we received alerts and were able to restart the service (after collecting heapdumps) and prevent an outage. We took our heapdumps and used the Chrome dev tools to start pinpointing where our memory leak might be. We compared three snapshots and found one of our internal libraries that we used for logging metrics holding references to log events. This helped us to reproduce the problem on our test system. In test, we generated some traffic that would cause the logging code to execute and used the new USR2 signal that node-heapdump added to dump the heapsnapshots every 20 minutes. We then used iptables to simulate a network outage to the downstream logging service. This gave us snapshots that showed lots of request objects being created and having references remain to these objects.

Because we extensively use feature toggles to enable and disable bits of code, we confirmed our suspicions by taking three snapshots over an hour with the module disabled, and three with the module re-enabled. With the module disabled, we saw no new objects created during our sample times.

The code that was leaking was pushing log messages to an array when the network request to post the message failed. There was code in place using promises to try to flush this buffer, but extended log service downtime would still cause this array to never be deallocated.

The solution to our problem ended up being to write failed log messages to our standard application log, instead of holding onto them when the log service on the other end was down. This is a similar approach to using UDP for sending syslog messages — think of logging metrics as “fire and forget,” rather than trying to capture every metric in a normalized event form. In the worst-case scenario (when the logging service is down,) we still capture the message and have our log aggregator slurp up the log message so it’s available via our dashboard. We can also alert on this as well.

Now that the dust has settled, here are some of our key takeaways:

* Measure, measure, measure! It’s ironic that the code causing our memory leak was code designed to measure. It’s important to ensure your measurement efforts aren’t causing a degradation in application or service performance.
* Toggle features on and off to help isolate big blocks of problematic code. This also helps integrate and ship code before it’s feature-complete.
* Beware the unflushed buffer! It’s a classic application problem that lurks in your code.
* Three snapshots helps you see a trendline. We followed the advice from the Gmail team and took three snapshots any time we ran an experiment on our test cluster to see a trend or eliminate noise.
* Blameless retros are much more effective at getting to the next steps and solving the problem (e.g. service degradation.)
* Heapdumps can be huge, so make sure you have enough disk space and are actively monitoring disk usage.

For more on blameless post-mortems check out [this post](https://codeascraft.com/2012/05/22/blameless-postmortems/) from Etsy’s engineering blog, Code as Craft. Here are a few good posts on diagnosing node memory leaks:

[Joyent on Walmart’s Node.js memory leak](https://www.joyent.com/blog/walmart-node-js-memory-leak)

[A post from Netflix about express](http://techblog.netflix.com/2014/11/nodejs-in-flames.html)

[StrongLoop on memory leak diagnosis](http://strongloop.com/strongblog/node-js-performance-tip-of-the-week-memory-leak-diagnosis/)

Shout outs to the Unicorn Bacon engineers: Ryan Farris, Matt Greer, Miki Rezentes, Joel Sherriff, John Skarbek, Patrick Winters

*Author's note - this post originally appeared in the Rallydev.com Engineering blog, which has been lost in the internet.  But thanks to the [wayback machine](https://web.archive.org), it is back online!*
