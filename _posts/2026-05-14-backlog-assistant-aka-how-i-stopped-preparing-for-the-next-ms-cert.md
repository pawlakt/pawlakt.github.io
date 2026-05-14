---
layout: post
title: 'Backlog Assistant aka How I Stopped Preparing for the Next MS Cert'
date: 2026-05-14
---

My native tech stack is built around the Microsoft ecosystem — Azure, Power Platform, .NET, all that stuff. So, in an attempt to somehow organize my AI expertise around the Microsoft stack, I started preparing for the AI—102 exam. I was actually pretty far into it when one day I came across an [article](https://techcommunity.microsoft.com/blog/skills-hub-blog/the-ai-job-boom-is-here-are-you-ready-to-showcase-your-skills/4494128) saying that AI—102 was basically being deprecated and replaced with a brand new set of shiny certifications.

With all due respect… these people are absolutely crazy with all the [naming](https://teybannerman.com/strategy/2026/03/31/how-many-microsoft-copilot-are-there.html), rebranding, splitting, and reshuffling of certifications 😄 I get it — AI is a massive topic and you can’t realistically squeeze the whole thing into a single cert. But with the current pace of development, I’d probably spend an entire year preparing for the next certification instead of learning something actually useful.

---

The best way to learn new things? Building things.
Ok then — what kind of things? Apps, automations, tools — honestly, anything useful. And since AI is supposed to be this magical super—tool now, why not use it to solve some actual problems?
Finding problems to solve is actually the easy part — most of us probably have an endless supply of them anyway.
One of my favorite ones: communication gaps. Most of them eventually turn into wasted time, confusion, or both. Another thing I kept noticing was work organization. Even in startup environments, where teams love to say they work “dynamically”, you still need at least some level of structure and order.
And honestly, many projects I’ve worked on had exactly this kind of pain around organization.

I asked myself — what’s the common ground between communication and work organization?
There are probably many valid answers, but for me it was the backlog. A document, dashboard, artifact — call it whatever you want — that should answer all the essential questions: What? Why? Who? When? How?
So I thought — why not use AI to help define and organize backlogs better?
At the same time, I realized that the startup I work in is running multiple initiatives in parallel, and with our very “dynamic” way of working, it’s super easy to lose track of things, miss requirements, or even completely forget about a product idea in the middle of the daily chaos.
So… free feedback & UAT (thanks guys!), and a chance to build something genuinely useful for the team? That was more than enough motivation for me to start building something backlog—oriented.

## How It Works

The whole idea is actually pretty simple — an internal, chat—based Backlog Assistant running as an Azure DevOps extension.
The workflow looks like this: the user discusses a project, feature, or even a single backlog item with an AI agent. They provide business context, requirements, random ideas — all the messy stuff that usually lives somewhere between Teams or Slack messages and people’s heads.
The agent is “smart” enough (#overhype 😄) to drill deeper into the topic and help explore it further. Since the app provides grounding behind the scenes, the agent understands the wider project context as well. For example, while discussing one feature, it already knows what other features are about and can help prevent overlaps or duplicated work.
The interesting part is that the agent constantly pushes the conversation forward. Prompt after prompt, it asks about requirements, actors, edge cases, non—functional requirements, and other details until it reaches a reasonable level of confidence.
At any point, the user can create a draft. Depending on how detailed the discussion was, the agent may generate a single backlog item or an entire set of them. From there, the user can continue the discussion, regenerate drafts, compare outputs, edit proposed items manually, remove redundant ones, or create separate discussion threads for specific backlog items.
And once the final structure looks good enough — the app simply creates the backlog directly in Azure DevOps.
One feature I personally find extremely useful is automated estimation. The agent provides rough delivery estimates right away, giving the team an immediate sense of how big or expensive a particular scope might be.

![Backlog Assistant flow](/assets/backlog-assistant-flow.png)

*Fig. 1. Backlog Assistant conversation-to-backlog workflow*

## Who Is It Actually For?

I originally started building this app with only one user in mind — myself. The goal was simple: organize my side projects and keep all my crazy ideas in one place instead of scattering them across random notes, chats, and unfinished TODO lists.
But once I showed the idea to my teammates, the reaction was surprisingly enthusiastic. That gave me even more motivation to keep pushing the project further.
After a couple of discussions about what should be improved, what would actually make the tool useful for the team, and where the rough edges were, we ended up with a pretty decent set of requirements — which, funnily enough, were turned into Azure DevOps items by the app itself during development 😄
Right now, Backlog Assistant helps our team validate ideas for new products, support grooming sessions for client projects, estimate implementation effort, and organize work around existing products.
I’d still call the project an MVP0, but even at this stage I’m genuinely satisfied with the results.
What surprised me the most is how much collaboration between a small team and an AI agent can improve the overall quality of deliverables. Better backlog structure, easier progress tracking, support with identifying missing tasks or edge cases, less manual backlog creation — all of that adds up pretty quickly.
And honestly, I’m pretty sure there are already enterprise—grade tools solving similar problems. I haven’t really checked. But I can absolutely imagine how powerful this kind of approach could become in large—scale projects with a proper engineering team behind it

## Technical Bits & Challenges

This is where things started to get real — and honestly, this is exactly where the actual learning begins.
To be fair, I don’t think I even spent an hour thinking about tech stack selection. I simply went with an Azure/cloud—native setup: Microsoft Foundry as the Agent/LLM interface, serverless Azure Functions written in Node.js for the logic, and the cheapest persistence layer I could reasonably pick — Azure Table Storage.

A couple of flashbacks from that part:

- I knew from the beginning that I wanted to build it as an Azure DevOps extension, but I wasn’t sure where exactly it should live. I first tried the hub/sitemap—like option, but eventually it ended up embedded as a separate tab on the backlog item.
- I’m slightly scared of Azure pricing — honestly, I wonder if there is anyone on the planet who actually knows what the real prices are 😄 So I started with the free GPT—4o mini model. I burned through the free token limit in one day of testing, wasn’t happy with the output, and eventually jumped to GPT—5. Turns out, for this kind of usage, it costs pennies.
- Azure Table Storage has limitations around the maximum number of characters per property, so I had to deal with that and do some partitioning. I know I could have used blobs or something else … doesn’t matter for now.

Then came the more interesting part — challenges around working with the LLM itself.
First question: how do you make sure your agent asks the right questions? And what does “right” even mean here?
I wanted the conversations to have a clear direction. The agent should not just randomly chat with the user. It should guide them toward useful requirements, missing details, edge cases, actors, non—functional requirements, and so on.
That’s where my prompting skills were properly challenged. To get the expected results, I had only one real option: test and evaluate. Over and over again 😄
Another challenge was the expected output. Should the agent generate the whole backlog structure at once — from Epic down to PBIs and tasks? I tried that, and the results didn’t satisfy me.
So instead, I changed the flow. First, the agent learns the raw idea of the app or system. Then it helps split the scope into meaningful, well—bounded features. After that, the user grooms those features one by one. At this level, the agent can generate PBIs and tasks with much better accuracy — and honestly, I was surprised how accurate those tasks were.
Estimations were another funny topic. How do you even

instruct an LLM to estimate tasks properly?
Should it estimate from the perspective of “an extremely efficient dev who crushes everything in a blink”? Or should I describe the team productivity curve hour by hour, including the post—lunch dip and the magical productivity spike right before 4 PM? 😄
I’m exaggerating, of course, but my goal was simple: provide estimates with roughly +/—20% difference compared to a developer’s estimate. Not perfect, but useful enough to support early discussions.
And finally — context management. It’s very easy to pollute the context with duplicated or unnecessary information. The output can change a lot depending on how much information you provide and how you provide it.
The same goes for grounding. How much project—level context should the agent know to understand not only the current backlog item, but the wider project scope as well?
All of this required a lot of evaluation. And because this whole thing is non—deterministic by nature, you need patience. There’s no magic shortcut here — you test, adjust, test again, and slowly move toward something useful.

## Lessons Learned

Turns out this well—known truth is actually true: building > reading docs and doing courses.

## Summary

At this point, I still treat this project as an MVP0. My main goal was to learn and get some real hands—on experience with this technology.
Mission completed!
And what’s even more important — our team ended up with a genuinely useful tool that helps capture ideas and transform them into structured backlog items.
The list of possible improvements is basically endless. To be fair, I’m actually surprised that Microsoft doesn’t provide something like this out of the box in Azure DevOps, because this kind of workflow feels extremely natural for backlog management.

## Reality Check

I know there are probably a hundred better ways to achieve the same goal — but honestly, that doesn’t really matter to me.
What matters is that I finally managed to build and finish a side project end—to—end. The legendary “Definition of Done” was actually reached 😄
Instead of jumping to a new idea every second day, I finally spent enough time with one project to properly explore it, learn a ton of new things, and — most importantly — have a lot of fun doing it.

You can check recent state of the project on [GitHub](https://github.com/pawlakt/BacklogAssistant)

NOTE: This repository is shared as a work-in-progress reference implementation, not as a production-ready product. It is intended to show the shape of an Azure DevOps Work Item AI Assistant and give others something they can fork, inspect, and adapt.

Before using it in a real organization, you should review and harden it for your own environment. In particular, pay attention to authentication and authorization, Azure DevOps token validation, CORS restrictions, secret management, logging of prompts/work item data, extension identity/rebranding, deployment permissions, and any organization-specific compliance requirements.

The sample configuration files use placeholders, and local `.env` / Function settings should never be committed. Treat the current repo as a starting point for experimentation rather than a drop-in secure deployment.

That’s it — stay tuned!
