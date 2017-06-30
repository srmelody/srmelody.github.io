---
layout: post
title:  "Regretrospectives and IGBTYOT — Musings From TriAgile 2015"
date:   2017-06-27 13:25:00 -0400
categories: agile triagile bdd
---



Post-event greetings from the Rally-sponsored Triangle Agile 2015 conference — a regional Agile conference based in the tech-heavy Research Triangle region of beautiful North Carolina. Rally sent several participants to the event on April 16, held at NC State University, to demo some of the great new products and features that we've released over the past quarters. I attended as a participant and picked talks that I thought were interesting. Here's my synopsis.

# Keynote

David Hussman was the keynote speaker — known not only for his passion around Agile but also for his past in an ['80s hair band](https://en.wikipedia.org/wiki/Slave_Raider) — and kicked things off with his band's music video. David noted that we're in the middle of a shift in thinking. The '90s were a decade of projects where as an industry, we were (incorrectly) certain about what we created. The 2000s were the decade of process with some ability to question what we were doing. Now, we're focused on products — we admit that we don't know what to build or what we're building, and we're guided by the goal of shipping product.

David remarked that it's not Agile to make every team have the same process. He also shared a technique called "regretrospective" where you can launch complaints, gripes, and problems from the team retrospective into the sky via a paper lantern. Obviously, you could extend this to write down ideas and burn them, or put them on a board of "things we talked about and are not going to worry about anymore."

# Tim Arthur — Nurturing the Journey: How to Train and Maintain Your Enterprise on Agile Practices

After the keynote, I heard Tim Arthur's talk. Tim is an IBM and SAS veteran, and discussed maintaining and growing Agile practices in a post-transformation environment. During his session, he underscored the importance of high energy! When you're on an Agile journey, it's important to start with high-energy training sessions and construct and maintain follow-up centers of practice excellence. You have to focus on basic knowledge, specialized learning, and continuous reinforcement.

Tim emphasized the need to continue learning from your data — not a training class, but continuous learning. When you're as engaged on your real project as you are on the training, great things can happen. Tim echoed a common theme of the day: how self-directed, self-organized teams are ideal. Your team is full of smart knowledge workers — they get it and don't need to be spoon fed or babysat. Tim also contrasted compliance with creativity. By avoiding compliance and standards, you can give teams tools and techniques to spark original thoughts and collaboration.

Overall, I liked the content of the talk and how Tim tied in concepts of learning through a small group exercise at the end. Attendees are more likely to remember what they learned because of the discussion afterward.

# Jared Richardson — Author of "ShipIt! A Practical Guide to Successful Software Projects"

Jared Richardson presented on Agile for managers and executives. Somehow they didn't check my architect credentials at the door and I got into the standing-room-only talk. One of the things I really liked was how Jared got back to basics — stop focusing on the Agile process checklist and make sure you grok the Agile Manifesto! Jared touched on the Dreyfus learning model . In an homage to a previous era of software documentation, he hinted that most agilists won't give you a checklist because they're afraid of the "redbook." Jared also proposed using a three-experiences approach (similar to [Brooks'](https://en.wikipedia.org/wiki/Fred_Brooks) advice to "plan to throw one away," reminding us that we need to consider code as disposable. Three experiences help you avoid a single prototype that becomes the project base.

I liked Jared's use of the term "greenshift" to describe how information radiates up the management chain and is spun in a positive direction at each step:

1. Go to the team — "How's the project?" "We're screwed."
2. Go to the VP — "How's the project?" "It's rough, but we're going to make it."
3. Go to the CEO — "How's the project?" "A-OK."

I've seen that before and it's critical to inject context into these information radiators, whether they're subjective or objective.

# Meeting New People

At lunch, I took the emcee's advice and sat at a table with people I didn't know. I was very fortunate to meet a Rally customer who told me that at her company, she highlights our use of feature toggles and feedback collection as an example of the right way to do software delivery. I explained to her a little bit about our feature-toggle system and how our feedback collection works toward improving our product and making our customers happy and productive. Rally's own Ryan Scott has written about the awesomeness of feature toggles in some of our most popular blog posts.

We also talked about shared challenges in refactoring a monolithic application into a loosely coupled architecture, and geeked out about the latest and greatest approaches to front-end development. Another table mate talked agility in a regulated environment and discussed how focusing small batches through his part of the system could be a way to improve delivery and still meet the rigid compliance needs at the end of his system.

# Ken Pugh

After lunch, I attended Ken Pugh's talk called "Effectively Communicating with Acceptance Tests." If you ever get a chance to hear Ken speak, make sure you do. Ken was full of energy and engaged the audience with examples of using acceptance tests and Fit tables as specifications for a set of features. I've used Gherkin scenarios on past projects and we're using JBehave on one of our projects at Rally. I think the key to leveraging Grid Driven Development (Ken's spin on BDD) is to know when and what to put in your Fit tables as opposed to other batteries of test suites. There has to be a trade-off between the ability for product owners and business analysts to specify requirements in Fit tables and the overhead of the construction and execution of these tests.

Ken also talked about the concept of marking tests with the expected result of "IGBTYOT," which means "I'll get back to you on that." Ken's suggestion was to fail these tests. I've also marked these as "pending" so they don't break the build but are called out as needing clarification.

![Ken](/images/ken.png)

I liked the focus on discovering ambiguity early and using tests as a record of our understanding. It's also important to note that acceptance tests are a point of collaboration — don't just throw these over the wall. Ken advocated using the "golden triad" as the group to build the tests before the story is worked by the team.

# Kanban Case Study

Leanne Gemma and Christine Woods from the McClatchy Company (a media group) presented a case study about their journey from Scrum to Kanban. I'm a big fan of Kanban because it helps teams always focus on the highest-priority and most-valuable work, and helps them control WiP and optimize flow.

They outlined the steps they took during their transformation to Kanban:

* Workflow mapping. Visualize the team's work on a board.
* Set WiP limits. You're probably hiding some process issues and bottlenecks; WiP limits help identify those. Two of their teams took different approaches to WiP: global (a max on the whole board) and column (each column has its own WiP limit).
* Set meetings and cadence. The teams kept standup and retros, resolving to take retros seriously. They removed sprint planning and design reviews, deciding on no set cadence to talk about what they were working on.
* Track metrics. Look at cycle time and ask how long it takes to get a ticket across the board. Why was that ticket so high? Was it blocked? Did we miss something? Are we seeing a crazy chaos of cycle-time variation?

Leanne and Christine reminded us that Kanban won't solve systemic org problems — as always, there is no [silver bullet](https://en.wikipedia.org/wiki/No_Silver_Bullet) . The presenters shared the good outcomes of the transition — fewer meetings and less conforming to artificial timeboxes that prevented teams from working on the highest-priority items. The challenges the team faced (lack of roadmapping being the biggest) are not unusual. Most Kanban teams that aren't entirely interrupt-driven will need to tie some of their queues to broader-level business objectives.

# When You Can't Do It All

Laurie Williams spoke about release planning. For input, it's important to understand the goal, velocity, and length, and choose whether you're embarking on a feature-driven or date-driven plan.

![Ken](/images/challenges.png)

Laurie highlighted the need to prioritize: 45 percent of development efforts go toward software that never gets used. She focused on a [Kano analysis](https://en.wikipedia.org/wiki/Kano_model) — mandatory, linear (the more the better), exciters (free water in a hotel room). She proposed surveying users and aggregating results around features and prioritization. Watch out for weasel words in your questions to avoid poisoning the well. She underscored the importance of relative weighting of cost to value (very similar to weighted shortest job first) when building your release plan. It's also critical to have a realistic plan: My team needs to have a fighting chance! With a release plan that is totally unachievable, I'll get less work out of it than if I made one that was sort of achievable.

# Andy Hunt and the GROWS™ Method

Andy Hunt delivered the closing session, introducing the GROWS™ method . His talk tied into the [Dreyfus model](https://en.wikipedia.org/wiki/Dreyfus_model_of_skill_acquisition) for skill development that was sprinkled throughout a few of the other breakout sessions.

Andy covered the motivation and main points of this new approach:

* All teams are different.
* What works for one team doesn’t guarantee it will work for others.
* What works today may not work tomorrow.
* There is no silver bullet  when it comes to Agile practices.
* Any new practice should be treated as an experiment and evaluated with empirical results.
* You should be implementing practices that are appropriate for the team’s skill level to get the best outcomes.

Andy requested attendees participate in and develop this new method together.
# Key Takeaways

It was exciting to see similar key themes and concepts woven throughout the day, including:

* Focus on delivering products.

* Have continuous-feedback loops, small-feedback loops, and fast-as-possible loopback.

* Make sure your teams are achieving learning and understanding, not just training.

* Look for signs of struggles around focusing on process and not on values (and re-read the Agile Manifesto).

* Use the golden triad (at the conference, it was dev, tester, customer).

* There will be lots of excitement around Kanban as systems mature.

The presentations are available [here](http://triagile.org/triagile-presentations/) . It was great to meet with customers, speakers, and fellow techie agilists. I look forward to next year's conference!

(Steven Boles contributed to this post)


*Author's note - this post originally appeared in the Rallydev.com Engineering blog, which has been lost in the internet.  But thanks to the [wayback machine](https://web.archive.org), it is back online!*
