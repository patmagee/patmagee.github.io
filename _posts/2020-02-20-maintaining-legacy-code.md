---
title: "Maintaining Legacy Code"
categories: 
    -   Software
tags:
    -   software maintenance
    -   obsolete
    -   code
toc: true
excerpt_separator: <!--more-->
---

At some point, every developer will be faced with the daunting task of being delegated to a "maintenance" role on the 
company dinosaur. Whether you were involved in the original project's inception, or if you are just stepping into the role now,
there are unique challenges (both internally and with the code), which come along with maintaining legacy code. Often,
there are lofty expectations about what can be accomplished in a relatively short period of time, with some not-so-lofty 
resources allocated. 
<!--more--> 
 
## How Did We End Up Here?

while it is very easy to point fingers at the "Developers of the past" as the ones solely responsible for putting you into
the current predicament, the reality of the situation is that we will all write software that is destined for a meager existence
as _legacy code_ at some point. The factors that lead to the decisions of the past are still often in effect today.

### 1. Shifting Internal Priorities

In the world of business and software development (especially with agile practices), priorities can change to meet market
demands seemingly overnight. When priorities change, new products are envisioned, old products are updated, and some products
are no longer needed. 

### 2. Over Engineered Software Bloat

One of the most common sources of legacy code is by writing code that was not needed in the first place. Often we try to over
engineer solutions attempting to anticipate all possible edge cases and pack as many features into a product as possible.
Unless features belong to the core competency of a product they generally will receive very little use and will rot as the
rest of the code base advances.


### 3. Technological Advancement

Another inevitable source of obsolescence is the steady march of technology towards ever newer and "better" frameworks, 
languages, paradigms etc. As new technologies enter the scene, they are adopted by developers eager to try out the latest 
and greatest. Sometimes these technologies persist leading to neglect of the "outdated" tech. Or, they are too eagerly adopted
and never actually catch on with the broader community. Either way, both scenarios will lead to a knowledge gap, and very
few people will even understand the old code.



## How to Work with Legacy Code

While you may maintain that the only REAL solution is to burn the legacy code at the stake, that likely will not be allowed
by the powers that be. In most scenarios, the only viable option is to work with the existing code strategically making modifications 
to be as impactful as possible. The key word here is __strategic__, and it is incredibly important that you adhere to this principal.
Failure to do so will inevitably end up costing you excessive development cycles and will **NOT** reduce the bloat or solve you problems.
Remember you are not trying to completely replace the product you are working on (if that was the case you likely would be starting from
scratch and not working on this monstrosity). 

### 1. Clearly Define Scope

Before starting any coding, it is imperative that you _clearly_ define the scope of what it is you are trying to accomplish.
Starting with the directive like "Making it faster" is simply too vague and will lead to rampant scope creep. Make clear objectives
defined by concrete action statements and issues, for example: 

- Improve performance of streaming by addressing known issue `#123123`
- Improve exception handling of business logic for `MainService` when transactions fail


### 2. Read Docs, Write Tests

Before diving into changing every aspect of the code, make sure you understand it! Understanding what the code does will help avoid duplicity and allow
you to address the real problems instead of making new ones. If good documentation was written that might be a good place to start.

Ultimately the best way to understand the code is to actually work with it, and there is no better way to work with code then to write tests. If there already
are tests (and I seriously hope there are), it is advisable to not just copy the existing ones but create entirely new ones. Test your assumptions about
how things work by writing tests about the behaviour you __think__ the code should exhibit. This process will likely lead to many fun and interesting surprises,
with comments such as "Why on earth would they do that", or "This really makes no sense" or my personal favorite "I have no memory of this place".

Once you have a firm grasp of the code (or at least the parts you are trying to change), you should shift gears and write tests for the behaviour
that you __want__ it to exhibit (if different from above). This will give you a definitive target that will help contain the scope of changes with which you can gauge:

1. Did you actually fix the problem
2. Are there any unanticipated side effects of the changes
3. Does your fix meet the criteria of marking the maintenance as "done"


### 3. Iterate on Changes

When you take the plunge and follow that rabbit down the hole, it becomes incredibly easy for scope creep to make it's way into your development cycle.
A change that should have taken a few hours now occupies the entire sprint and other projects now are relegated to the back burner. I like to think of this as the "never ending sprint", since in every standup update you say something similar to  "Still struggling through the code", or "It will be fixed... soon",
or "I think I am getting close". These are clear indicators that you are in a non-progressing dev cycle which makes it impossible to determine actual progress or project completeness. Remember the goal of maintenance is not to rewrite the product, but to work **with** what is already there.

The [scope of changes](#1-clearly-define-scope) you or you project manager defined should easily fit within a single sprint, or even a single day to allow for rapid iteration and incremental fixes. Each iteration should never leave the code in a broken state. This will easily lead to stale code if you get pulled away from what you are working on for a time, but it also makes it difficult to stop working on the project. If the powers that be decide that you have done enough, you should simply be able to walk away at the end of a sprint, knowing that the code is a little bit better then when you started.

### 4. Refactor, Don't Reinvent

Refactor the code whenever possible, and avoid changing any functionality even if you _know_ you can do it _so_ much better. Refactoring is the process of changing the structure of the code, improving it's readability and performance without ever changing what it actually does. It's often difficult to determine everything which depends on a specific functionality, even if you do not see an inherent need for it. Everything likely was created for a purpose and serves a specific need that your teammates encountered in the past.

Additionally, now is not the time to be trendy. When you refactor, the emphasis should not be on including the latest and greatest paradigms from the fringe niches of the development world, but should favor consistency with the current code base and cleanliness.    

### 5. Up the Bus Factor

As a codebase ages and goes from recent memory into myth, and then myth into legend, there seems to be one universal truth: all domain expertise over that code base is lost. It's not entirely uncommon to have a legacy project in production that no one knows how to maintain. This situation can arise when all or most of the original developers have left the company, or if the code has sat for many development cycles without ever being touched.

It's tempting to try to limit the amount of pain and suffering experienced by the team by restricting the number maintainers to one, or at most two developers. This may seem like a logical choice, since everyone's time is important and the more developers you have working on maintenance, the fewer you can have working on the new flashy products. This trap will inevitably lead to **more** time being wasted re-learning the code with each employee turnover. The key then is to increase the bus factor, and make sure that the knowledge of how to work with the code is distributed amongst multiple people on your team. Some tactics that have helped me in the past are:

- After you have taken the deep dive into the code, share the knowledge! Hold a lunch and learn to explain to your coworkers what you have discovered and lead them through the code base.
- Do some pair programing to educe others on some of the more difficult portions of code that encountered
- Take turns maintaining the code. Have one person address one or two small issues within a sprint and then hand off to another person in the next sprint.
- Document any hurtle you had to overcome so that the next person who works on it will know exactly what to do.
  
## Conclusion

Maintaining legacy code can be an incredibly daunting task, but it does not need to be difficult, nor does it need to consume all of your time. Following the five tactics listed above will help you effectively maintain even the oldest codebase with the least amount of impact on your overall productivity 


