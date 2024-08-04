---
title: How Big Things Get Done
keywords: Book Summary, Project Management, Management
date: 2024-08-04 18:47:13
tags:
  - Development
  - Summary
---

If you’ve been in the industry for more than a few years, you might have seen at least one big project that failed badly - maybe it went significantly over budget, had its deadline repeatedly pushed back, or it delivered such a buggy product that users went back to using Excel. 

Usually when people talk about such projects in casual settings, they would attribute the failure to incompetent leadership. As you grow in your career, you will slowly get asked to lead bigger and bigger projects. How do you avoid being that "incompetent leader"? Actually, what does competent leadership in this context look like? What do they do differently that make projects successful?

In their book [How Big Things Get Done](https://sites.prh.com/how-big-things-get-done-book), the authors have researched what differentiates successful projects from the failed ones. 

As the saying goes, [*All happy families are alike; each unhappy family is unhappy in its own way*](https://en.m.wikipedia.org/wiki/Anna_Karenina_principle). Similarly, according to the book, all successful projects exhibit the following attributes:
* Understanding Motivation
* Thorough Planning
* Forecasting Reliably
* Prototyping Iteratively
* Valuing Experience
* Building High-Performing Teams


## Understanding Motivation

Most of the time, users don't know what they want. So it's important to ask questions at the beginning of the project to understand why they need something built, their fears, and their aspirations. Since projects are just vehicles for achieving some outcomes, by understanding the project's goals, we can come up with better solutions.

If you don't have users yet, it's better to start with a short summary of the product, why it's valuable to the users, and then work backwards to make sure the product fits in with the initial summary.

## Thorough Planning

### Two Systems of Decision Making

In the book [Thinking, Fast and Slow](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow), Daniel Kahneman says that humans have two decision making modes - "System One" and "System Two". System One is the default mode. It is when we make snap judgments based on emotions or intuition, and this is how everyone operates in real life. This works great when done by experts as they have years of experience to draw upon. System Two is about conscious reasoning, where people correct the decision made through System One or even override it.

### Two Patterns of Project Planning

This concept can be applied to how projects are planned and used to categorize the projects into either "Think fast, act slow" or "Think slow, act fast" patterns. 

In the first pattern, people don’t spend too much time on planning and rush into development. There is rapid progress initially and everyone is happy. But because of superficial planning, we fail to uncover the problems and come up with potential solutions. So when the project hits these problems, it comes to a standstill. The project devolves into a mess.

The second pattern is the opposite. By doing thorough planning, we can get a realistic understanding of the timeline, costs and potential problems. We can then come up with solutions for those problems, change the scope of the project or just cancel it altogether if it didn't make sense.

### Why Planning is Skipped

System One thinking is not the only reason people avoid thorough planning. Sometimes it's skipped for malicious purposes. For example, some might lowball the initial estimates so the project gets approved. Others might rush the project so no one can stop it because of sunk-cost fallacy.

## Forecasting Reliably

Forecasting errors is another reason why big projects fail. 

### Anchors and Adjustments

People usually base their estimates on how long similar projects in the past took. If they haven't done a project of the same scale, they would use the estimates for a smaller project. This is called an "anchor". They would then "adjust" these estimates for the bigger project. This is a natural way of thinking, but anchors are tricky and using wrong anchors would yield wrong estimates.

### Reference-Class Forecasting

All projects are unique in their own way. Maybe it's the team, maybe it's the economic situation, or it's some unique combination of such factors. However, projects are not 100% unique. There would still be large parts of the project that are comparable to other projects in the same class. This is called a "reference-class". It makes sense to use these already completed projects that are in the same class as our project. We use them as anchor and adjust the estimates based on how much our project differs from the mean in the class. The downside of this approach is that obtaining the numbers for the reference-class might not always be possible.

## Prototyping Iteratively

### Why Experimentation is Necessary

Planning is usually associated with images of endless meetings, bureaucratic paperwork and approval flows. However, these things are not the only aspects of planning. Experimentation and iteration are also part of planning as they help us come up with a detailed and reliable plan. If the project we're starting has some unknown or risky elements, it's better to experiment with some prototypes or proof-of-concepts (POC), so we know that those risky elements are solvable.

### How Iterative Planning Works

Iterative design is also considered planning. If we split the project phases into "planning" and "development", we would often notice that "development" is the most expensive part of the project. Whether it's constructing a building, or producing a movie, etc. Whereas "planning" is relatively cheap and safe. We can iterate as many times as we want planning the building construction in CAD or storyboarding movies. The design starts with low fidelity and as it progresses, it gets more and more high fidelity. 

For example, each Pixar movie starts with an idea, develops into a 12 page outline, then to a 120 page draft, to a detailed storyboard, to internal movie screening, and then finally to the world. Each stage goes through multiple iterations, and the script gets substantially rewritten and a good portion of the work done in initial phases are discarded. 

While iterating like this is expensive, it gives us a lot of advantages:
* It gives freedom to the team for experimentation, to keep what works and discard what doesn't.
* It ensures that every part of the project gets scrutinized and tested before the development starts.
* It makes sure that the team truly understands the problem before starting work.
* It is cheaper to throw away mid-planning than during mid-development. 

In the software world, releasing an MVP (Minimum Viable Product) is also considered planning. It's the act of actively experimenting with the product and releasing it early to see if it is what customers want.

## Valuing Experience

People, processes, and technology can have "experience". A person can gain experience by spending a lot of time perfecting their craft. A process, technology or tool can also "gain" experience or become "proven" by getting iteratively refined. Trusting experts or using proven tools feels like common sense, but this needs to be said out loud.

### Experienced People

Experts know more, though there are two types of knowledge - "explicit knowledge" and "tacit knowledge". Explicit knowledge can be obtained from books, classrooms or other sources, whereas tacit knowledge is the skilled intuition that only people with long experience have. 

*Sidenote: Big projects have a lot of moving parts like multiple teams, stakeholders, agendas etc, so a leader with more experience doing such projects would be able to navigate all these intuitively and make the project a success. This tacit knowledge gained from experience is not fully "serializable" to books or howto manuals (explicit knowledge). Though a novice who obtained explicit knowledge from books would be still better than one who has zero knowledge.*

Experienced people often get aggressively marginalized or dismissed because leaders don't appreciate how well it helps with project planning and delivery.

### Proven Technology

Technology or systems can be thought of as "frozen experience". People choose newer technology because they assume newer is better or they value custom-made or bespoke technology as they're "unique". But these are just "inexperienced technologies".

*Sidenote: This is similar to the [boring technology](https://mcfunley.com/choose-boring-technology) movement we've been seeing lately. This is where developers choose mature components like Postgres, Server Side Rendering, Monoliths, etc instead of whatever the current hotness is.*

## Building High-Performing Teams

Planning is very important for a project's success, but a capable team implementing the plan is equally important. Big projects by nature often require people from multiple teams or external vendors work together to deliver it. 

### Conflict Resolution

It's very common for conflicts to arise when multiple companies or teams are working together to deliver a project. When there are conflicts, it's important for the leadership to intervene quickly and defuse the situation so it doesn't get worse. Even better if contracts are designed to reward positive outcomes and to make sure that it's in the interest of the vendors to work together to achieve a common goal.

### Identity and Purpose

It's important for people from multiple companies to feel like they're working as a single team for the project instead of being employees of respective companies. Equally important, it's for people to understand the outcomes of the project and why their work matters. These need to be said repeatedly for people to feel a sense of belonging and purpose as a single team.

### Open Communication

When leaders listen to the team's feedback and quickly make the changes, it creates psychological safety. It improves the team morale, increases improvements, and problems are identified and fixed quickly.

None of these are easy. We need to invest a lot of time, effort and money to develop the team dynamics, but it makes a huge difference in project success.

## Conclusion

As we can see, a lot of these ideas are either straightforward to implement or considered common sense. What sets successful projects apart is how well these are executed. Once you know these ideas, it's hard to unsee them in failing projects.

