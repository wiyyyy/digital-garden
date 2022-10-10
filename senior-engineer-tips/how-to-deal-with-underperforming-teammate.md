---
tags: senior-engineer career team
---

# How to deal with an underperforming teammate

## Question
I have a teammate, we'll call him Jerry, who I've been having trouble with at work. This is Jerry's first dev job, but he's been at the company for about 2.5 years and on my team for 1.5. After a year and a half, he still asks me and other devs on our team (most of whom have less experience than him) very basic questions about how our only product works. He is mostly given unimportant tasks that he takes weeks/months to finish while asking tons of questions and scheduling one-on-one meetings with other devs on the team. At the end of this process, he shows up with something that's borderline unusable and fights tooth and nail to avoid making any changes during code review. He's completed two projects since joining, one of which had to be completely rewritten by a more junior dev while he was pulled away for budgeting reasons to help another team implement something using open source tool X. The other is a testing suite that is mostly ignored and has multiple failing tests in our master branch because it's painful to deal with.

​

My solution to this is to try and avoid Jerry as much as possible, so I'm not the person he's scheduling meetings with or downstream from his work. Recently though, after pushing for quite a while, I got the green light from my manager to start working on a solution for our team that involves X. I was told it wasn't a high priority for the business, so I wouldn't be receiving any help from the rest of the team. But, after Jerry directly asked me, my manager, and my scrum master multiple times to help with the project, my manager agreed. I wrote him a straightforward and relatively small story to take an existing feature from our product and implement it using X. As I expected, it's now almost 4 weeks later and despite all of the one on one meetings and DMs we've exchanged he's not finished and the solution in his working branch is way off the mark.

​

At this point, I'm beyond frustrated and I'm finding it hard to stay professional when communicating with him. I don't want to get the guy fired, I just can't stand having my time wasted by him. The trigger for this post is when I checked my email and saw that he scheduled yet another meeting (this time before my working hours) to discuss not only the story we've talked about plenty already, but the other, unrelated task he's assigned that he also can't figure out by himself.

## Answers

I’ve been in this exact situation before and I’d say it rarely ends well.

My solution was to try and build up my Jerrys skills by cherry picking easier tasks and slowly building up to bigger features and more complex tasks, trying to include the person in planning and design talks etc.

What I found was that no amount of PRs, comments, advice or handholding can help someone who is just there to collect a pay check and either doesn’t want to build their skillset or doesn’t care about the work.

I fired my Jerry after trying to help them improve for way too long. The result that every time we found something in our codebase that was wrong, over complicated, or just designed in a stupid way, a quick review of the history always pointed to Jerry.

So the issue you have is these type of people don’t just soak up your time now, they also take it long after they are gone while you’re still fixing their stuff.

If I were to be in your situation where I’m not the manager I’d first try have a talk with the person in a less formal way and bring up that they seem to be struggling a lot / asking a lot of questions, and is there any problem that you could help with.

Obviously that might not be well received, but I think it’s the right thing to do before escalating. Also they wanted to be on your project so try find out why, maybe they wanted to use it as a learning opportunity.

Then you need to gather some evidence and go to your manager and show how this kind of work is bringing the team down and your actually losing productivity by having the person on your team.

It sounds like this person needs a lot more training and handholding to improve their skillset. One way you might go about this is more pair programming with them and discuss how to implement things and the downsides to each idea etc so it’s more of a learning experience.

Also they should be held accountable in daily stand ups. If they are working on something and it’s taking way longer than it should the question needs to be asked why.

However it does sound like part of the problem (in a small way) is the team. You have someone you know isn’t performing and yet they get put on projects solo. That just sets them up to fail and also doesn’t give opportunities to learn. They will also probably fight PR comments as the thing being criticised is a huge bit of work with like a month of effort so They will naturally be more protective over it and probably just want it done if it was a struggle to get it this far

[https://old.reddit.com/r/ExperiencedDevs/comments/tcc512/how_to_deal_with_an_underperforming_teammate/](https://old.reddit.com/r/ExperiencedDevs/comments/tcc512/how_to_deal_with_an_underperforming_teammate/)