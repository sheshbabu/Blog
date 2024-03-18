---
title: Thoughts on the Future of Software Development
description: My optimistic take on the future of Software Development in the age of AI. 
date: 2024-03-18 17:57:18
image: /images/2024-thoughts-on-the-future-of-software-development/2024-thoughts-on-the-future-of-software-development.png
tags:
  - LLM
  - Development
---
Large Language Models (LLMs) caused a huge stir in the creative circles when they were able to generate images, text and code. Initially the results were quite hilarious with [drawings of people with messed up hands](https://twitter.com/weirddalle/status/1617240690290839553), hallucinating incorrect facts and code. But things are slowly *and steadily* getting better. Before the advent of these models, the main argument against automating these tasks was that machines can’t think creatively. Now this argument gets weaker by the day. Where do we go from here?

The downside of trying to think about some vague problem like predicting future is that your thoughts get muddled and it’s hard to think clearly. So we need to come up with frameworks and analogies for us to lean on. 

![](/images/2024-thoughts-on-the-future-of-software-development/2024-thoughts-on-the-future-of-software-development.png)

## Framework: Software Development Capability Level

Software development is not just about writing code. The image people have of programmers is a person sitting in a dark room looking at computers and furiously typing code. Though coding all day sounds very appealing, most of software development time is spent on communicating with other people or other admin work instead of just writing code: 

- Gathering requirements from business users
- Refining these requirements so they can be modeled as code
- Talking with other team members like Designers and Product Managers to visualize the solution and coming up with a plan of attack
- Working with other developers to come up with a technical design and refining it
- Setting up infrastructure, configuration, boilerplate etc.
- Actually writing some code
- Debugging, trying to understand other people’s code, writing documentation, etc.
- Deploying to production
- Firefighting production issues
- … and so many more tasks

So saying things like “AI will replace Developers” requires “AI” to be competent in all the above tasks and not just writing code.

> So saying things like “AI will replace Developers” requires “AI” to be competent in all the above tasks and not just writing code.

But looking at the list above, it looks like some of these tasks can also be automated in future but not yet. How do we frame this thought? 

The world of self-driving cars come up with a way to [classify the level of automation](https://en.wikipedia.org/wiki/Self-driving_car#Classifications). It has discrete levels and goes all the way from no automation to partial automation to full automation. I find this very useful for many reasons:

- It clearly describes what the current technology is capable of
- It prevents us from thinking in black and white - it’s not about human driver vs AI driver where AI fully replaces human drivers, it’s possible to have gray areas where human drivers are assisted by AI in things like emergency braking, lane centering etc.

How does such classification look like to AI driven software development?

- The lowest tier would be what we had before - no AI involved in work. Of course we had other types of automations like compilers, build processes, etc., but these are not AI, these were human written deterministic automation.
- The next level is what we have now - developers using ChatGPT or GitHub Copilot to assist them. They use it for things like writing tests, boilerplate code, refactoring, understanding code/errors etc. It’s like talking with a fellow developer over chat whom you can ask questions and get some help from, but they don’t have access to your machine so they can’t create files, run build commands or deploy to production.
- The highest level would be like delegating part of your project or the entire project to a developer. These “AI coders” would take in the requirements, write the code, fix errors and deploy the final product to production. I assumed were still many months away from this happening, but was proved wrong with the [Devin demo](https://www.youtube.com/watch?v=fjHtjT7GO1c) - even though it can only [perform simple development tasks](https://favtutor.com/articles/devin-ai-early-insights/) now, there’s a chance that this will improve in future.

![](/images/2024-thoughts-on-the-future-of-software-development/2024-thoughts-on-the-future-of-software-development-01.png)

Apart from what the AI model is capable of, we should ask think in terms of how accurate the solutions are. Initially these models were prone to hallucinations or you need to prompt them in specific ways to get what you want. This adds friction to adoption and most people give up on AI assistants at this point. But this is also improving, the newer models don’t need that level of prompt engineering. Also, the models should be able to “learn” by browsing the web instead of relying on their training data. This is important as new versions of libraries and programming languages get introduced. 

## Framework: Outsourced Software Development

Now that we’ve established the capabilities, how would these influence the team or organizational structure? Companies do software development in multiple ways:

- Fully inhouse
- Mostly inhouse with few vendors
- Mostly vendors with few inhouse
- Fully vendors

![](/images/2024-thoughts-on-the-future-of-software-development/2024-thoughts-on-the-future-of-software-development-02.png)

In a way, we can think of AI coders as outsourced software vendors/consultants. Some companies use them a lot and some don’t as much. Irrespective of the composition, I believe it’s always important to have an inhouse team oversee their work. This is to make sure vendor’s output is aligned with your organization’s long term goals. Of course, you can solve this via contracts, but they usually only apply to a specific vendor or a project, and you can’t enforce long terms goals using this method. It’s always better to have at least a small inhouse team who can guide the vendors. Similarly, even when AI coders can be rented out like EC2 instances, it will be beneficial to have an inhouse team of Software Developers to oversee their work.

## Framework: Software Development Is Modeling Complexity

If we’re talking about solving business problems, let’s take sometime to talk about the elephant in the room - Excel. It’s a well known secret that the world runs on Excel, and [more than 1 Billion people](https://news.microsoft.com/speeches/satya-nadella-and-terry-myerson-build-2016/) use it. It provides a very low barrier to entry for business users who want to organize data, perform data analysis, or automate some process. However, we can’t use Excel for complex business workflows as it doesn’t have features like granular access control, ability to integrate with unsupported systems, testability, reusability, or just vendor lock-in etc. The same can be said for “Low Code” solutions like [Power Automate](https://www.microsoft.com/en-us/power-platform/products/power-automate), etc.

Coming back to the original question, would business users be able to use AI coders to create these complex workflows without the help of software developers?

> Would business users be able to use AI coding tools to create these complex workflows without the help of software developers?

If you think about it, Excel and Low Code tools have existed for many decades, so why does the Software Development profession still exist? It goes back to thinking of Software Development as just writing code. For complex problems, we need people who can effectively manage these complexity and translate the business problems from real world domain to digital models. 

In other words, if you’re able to build a wooden shed from YouTube tutorials without the help of a Civil Engineer, doesn’t mean you can/should do the same for a 10 story building. If you go about learning how to do this properly, then you slowly become a Civil Engineer! It’s just a matter of whether you’re willing to put in the time to learn this properly or hire an experienced Engineer to do it for you. 

So whether these people are using Excel or the latest AI coder, if they’re modeling complex logic, they’re still software developers in my opinion! They’re just using different tools to express the business requirements - spreadsheet formulas vs code vs prompts. 

## Framework: Size Of The Pie

Most of the anxiety surrounding this topic assumes the size of the market for Software Development remains the same - AI coders will slowly take “market share” away from humans. 

![](/images/2024-thoughts-on-the-future-of-software-development/2024-thoughts-on-the-future-of-software-development-03.png)

From the previous section, we know that the market size of “solving business problems” is much much bigger than just Software Development. So there’s no reason to believe that Software Development will disappear anytime soon.

![](/images/2024-thoughts-on-the-future-of-software-development/2024-thoughts-on-the-future-of-software-development-04.png)

## Framework: Formal Business Logic Definition

Business logic must always be defined in an unambiguous format. This is why programming languages, even though they use English words like “if”, “switch” etc., are very particular about what these words mean and won’t work if you use the wrong words. If you think about it, the same goes for Excel formulas or Low Code flows. 

In future, even if the AI coders could generate a software product from instructions given in conversational English, I believe there would still be an underlying formal definition of the business logic generated in the backend. It might look very different from the languages and frameworks we use today, but a formal definition of business logic sounds a lot like “code”. 

Until AI coders can start generating these business logic from conversational English in a deterministic manner, there would still be a need for people who can understand the code it generates in the backend and make changes if necessary. These people would be Software Developers. 

## Conclusion

In summary, I believe there would still be a market for Software Developers in the foreseeable future, though the nature of work will change and the tools we would use might be very different from what we have now.