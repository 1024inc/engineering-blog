---
title: Post-mortems intro
author: Ines Martins
date: 2022-02-02 11:33:00 +0800
categories: []
tags: []
math: true
mermaid: true
image: /assets/images/posts/post-mortems-intro/skeleton.gif
src: /assets/images/posts/post-mortems-intro/skeleton.gif
width: 800
height: 500
---

<h2 data-toc-skip>Background</h2> <!-- testing github webhook -->

In the past, one of the Beyond engineering core values was “We Ship It!” More recently, we translated it to "Ship Early." At Beyond, the best way to ship safely is to ship often, and the best way to ship often is to ship smaller.

Shipping early is a good way to get feedback, to quickly iterate and improve on code, break wrong assumptions, rearrange logic and fix unexpected (hopefully) small errors…

We work towards perfection, but we realistically know we will never be 100% perfect, not all the time. Releases do not always follow the happy path; something unexpected may happen and then we honor the other Beyond engineering principle, "We Take Ownership." We jump into action and quickly revert or fix those errors.

![Desktop View](/assets/images/posts/post-mortems-intro/ship-it.gif){: width="400" height="200" }
_“What went wrong, and how/what do we learn from it?”_

If we take notes during the “fixing process,” from the time we found out something was wrong and what we have done to make it right, we can build a post-mortem!

The concept of “post-mortem” originates from medical jargon. By definition “it is a surgical examination of a dead body in order to find out the cause of death.” This analogy can be transcribed to the engineering field as a detailed examination of an incident after it has happened. It is a useful tool to describe major incidents for the rest of the team to have a clearer picture of what went wrong and what was done to fix it!

Post-mortems are intended to help us learn from past incidents and mistakes to avoid repeating them in the future.

The value of post-mortems fits perfectly with Beyond’s culture since we advocate for everyone's continuous improvement! We embrace our mistakes and learn from them!

Other companies like Google, Facebook, Amazon, and Github have also adopted this type of documentation. [**Here**](https://github.com/danluu/post-mortems) you can find a github repository with an extensive list of post-mortems from those companies.

<h2 data-toc-skip>What are post-mortems and why are they valuable for Beyond?</h2>

As Beyond founder and CTO David Kelso once said: “It's never fun when mistakes (...) happen, but what really matters is how we handle them.” At Beyond we do not point fingers, we do not blame people if they make mistakes, because in the end we all are humans and humans are very error prone!

While someone is fixing a bigger issue, they are 100% focused on putting things back on the right track as soon as possible. They cannot (and should not) be wasting time and mental energy thinking about optimization or performing a deep dive on what caused the incident, we just want to get everything right very quickly! A good practice we tend to follow here, is to take notes during the problem solving because this will help to build great post-mortems that will be essential to provide an opportunity to reflect once the issue is no longer impacting users.

Without a post-mortem, we may fail to recognize what we were doing right, where we could improve, and most importantly, how to avoid making the same mistakes in the future and come up with action items for a better long term solution.

Still we need to emphasize that post-mortems are optional at our company. They are internal only, more targeted to engineering, but can also be really helpful for other departments like support and customer success managers to understand what happened and to help them polish their public messaging to our affected customers.

![Desktop View](/assets/images/posts/post-mortems-intro/1J37vylrl9IGdS2iwuWo4KZHe1Tt_Ocli6rxWp4k.png){: width="400" height="200" }
_“It's never fun when mistakes (...) happen, but what really matters is how we handle them.” - David Kelso_

<h2 data-toc-skip>Basic Structure of a Post-mortem</h2>

These are all the areas that we explore with a post-mortem at Beyond:
- Overview - should be a super short 1 or 2 sentence description with quick reference to timeline, what happened and impact - consider this a “TLDR” section
- Background - If applicable, add any required background or context that is worth sharing for understanding the big picture of the problem
- What happened - Which servers/applications were affected? If any of the measures we took to solve the problem make it worse, we should describe them here also.
- Root Causes - Description of root cause for the Incident. It can be a superficial description if we are still not sure about root causes, or an in depth explanation if we are certain about the root causes.
- Mitigation - Basically answer the questions: “What was done to fix the issue?” and “How long did it take?”
- Impact - This is really a very important analytical section to show the damages and the impact on the business, for example: number of users/listings affected, syncs failed, if it was a billing related issue show values breakdown, etc. We can link third party media here, like a mode report or screenshot of datadog charts, logs, etc.
- Responders - Name of the people who were involved with fixing the issue
- Timeline - From start to finish, we can detail actions attached with the time they were taken. Try to focus on the most important facts that happened; no need to have all the little details, but if you have them, you can be as specific as just putting a date or a date and a time.
- Good-jobs and Improvement Chances -  Answer the questions: “What did we do well?” and “Where's the chance to improve?”
- Action Items - if a quick fix was applied, try to propose and follow up with a more structured long term optimized solution.

![Desktop View](/assets/images/posts/post-mortems-intro/skeleton.gif){: width="200" height="200" }

<h2 data-toc-skip>How to write a Post-mortem?</h2>

Humble advice on how to write a (fairly good) post-mortem:

1. Once you detect that there is a problem, keep calm… and start to take notes! (Of course you can just rush to make everything right fast but have small bullet points of what you have done to make things correct again, writing them down will help to build the post-mortem once the problem is fixed with peace of mind)
2. Do not forget to populate the incident timeline with the most important changes in status and key actions taken by responders.
3. Analyze the incident: describe superficial or in depth causes as much as you can… Tell which systems/servers were affected, was it a front or backend issue? DB? etc..
4. For the impact part: it is nice to include charts or any other type of metric or some third-party page where the data came from. Bonus: if you can prove or illustrate that the “recovery” was successful.
5. Create Jira tickets that may result from action items for a better long term solution.
6. Write full body text, polish the messaging, and ask for a review from one of your colleagues or from your manager.
7. Share the post-mortem.
8. Sharing is caring! You should not feel shame - share with your teammates so we can all learn from mistakes to avoid them in the future!!!

![Desktop View](/assets/images/posts/post-mortems-intro/1o3Ol7Wdw0QSqjRHaIZIQ1fZbRK0LL2lBnaUxTMk.gif){: width="400" height="200" }
