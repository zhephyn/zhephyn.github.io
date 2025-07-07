---
layout: post
title:  "Why Junior Rails developers should contribute to Open Source projects"
date:   2025-07-04 21:49:09 +0300
categories: ruby
---

## The path to land a tech job is documented online. 
Before the tech job market became a mess with AI and big tech lay offs, the path to becoming a software developer was pretty straightforward. Build projects, make a portfolio and then apply to jobs. This was applicable whether you're self taught or whether you went to a boot camp. Most self taught developers would choose a tutorial in their language of choice and build along with their instructor. After working through this tutorial for whatever duration of time, at the end, they would have a project to show off, an end product of having worked through said tutorial. 

Nowadays you need more than a list of tutorial projects you've worked through to land a soft ware job especially for junior developers. And as someone trying to land your first tech at arguably the worst point in time in tech history, you'll have to do a little bit more than building projects to stand out from the crowd. My personal experience working through online tutorials led me to the conclusion that, "Tutorial projects are not enough. Supplement them with Open source contributions". 

Through out this blog post, I have one aim, to convince the fellow junior developer reading this to consider contributing to open source. 

## How Open source addresses the problem with programming tutorials.
The problem with most tutorials is that even after you've done and completed many of them, you've done a lot of copy pasting along the way and you don't really understand why the creator of the tutorial chose the syntax they ended up with and more importantly, if there are better solutions to the problem at hand. Most tutorials focus on writing code that just works, and most junior developers including me, "We don't know what we don't know". This is where open source comes in as a solution to the tutorial problem. 

For a junior developer, contributing to open source is comparable to being thrown in the deep end. You'll have to navigate the code base your self, you'll struggle with a lot of bugs that are introduced by your new code, failing CI tests and so much more. Every line of code you write has to be accounted for and tested that it won't break existing functionality. If the code can be rewritten or refactored, you'll be instructed to do just that.

> ***"Every code base has a given standard of code quality and for your PR to be 
> merged, you'll have nothing to do other than meet that standard."***

The good news through out all this is that, **by the time that PR is merged, you'll be surprised at the sheer amount of stuff you didn't know when you started out**.  

![Kafka on the shore quote](/_site/assets/images/haruki-murakami-kafka-on-the-shore-quote.png)

## My first Open Source experience. 
In the last quarter of 2024, i made my first open source contribution to a project called **Avo**. My thought process about how I would work through the assigned issue was naive in every sense. I imagined that i would create a new branch for the issue i was working on, make changes to the code to implement that feature, go to localhost:3000 in my browser to see if the feature works as expected, then push the PR and have that PR merged to the main branch. 

Of course that didn't go as i had planned it out in my head. First and foremost, I realized that making changes to the code base as i went about implementing the feature brought about a few things. New bugs were introduced, i ran into errors and the new code caused tests that were passing on main to fail on my branch. 
Which brings me to my first lesson, "For every code change you make, appropriate tests should be written for that code" and "testing is the only way for you to be sure that a feature works as expected." A failing tests definitely highlights that something is wrong and a passing test gives you confidence that the feature implementation was successful. Having experienced this ordeal, i appreciated writing tests more and the ideas behind test driven development commonly known as TDD. 

**My new code caused CI feature, system and lint tests to fail**:

Secondly, when you push a branch to the PR, the project CI tests your code against all versions of the project as many projects tend to offer support for old version of projects. In the rails ecosystem, this means that even if the current code base is powered by the latest version of rails i.e. Rails 8, the code you introduce should work okay even on previous versions of rails, both major and minor versions. This includes Rails 6, 7, 6.1, 7.2 and so on. "The code you introduced should not break any existing functionality or tests". Therefore for your code to be finally merged, all checks ran against that PR should pass. 

There are other lessons i learnt of course that are related to the CI for example working with code standard tools such as Rubocop, making sure that your changes work on both new and major versions using a tool called Appraisal from Thought bot and so on. The main point I'm trying to put across is that, you don't know what you don't know and the amount of stuff i learnt just by working on that first PR was so enormous. 

## Benefits of contributing to open source

1. Making it work is the first step, making it maintainable and making sure that the code you've written meets the standards of that code base is the second step. By contributing to open source, you go from "this code works" to "Is this code maintainable in the long run? and if future me was to implement a feature that relies on the code I'm writing today, how many and what kind of changes would he have to make to this code?"

2. Its a hands on lesson in refactoring. Most times when you read books, in my case, Ruby programming books, they show you a piece of code and then give you ideas on how that code could be refactored. While this is good, for such refactoring knowledge to be ingrained in your memory, you'll need to put it into practice.          If you're working on a PR and you run into a use case where one of those refactoring ideas may apply, you'll appreciate the refactoring more simply because this time around, you'll have context and you'll be able to answer questions like: Why did we choose this specific idea to refactor this code and not the other?, What are the implications of this refactor to other parts of the code base?, Why is this refactoring technique better than the other? This shared context is very important because after learning it for the first time, the process and technique you applied to perform that refactoring will be with you for life. In my experience, refactoring ideas and techniques are best learn from experience. You need to make your hands dirty.

 **Refactor to use meta programming instead of if statements as part of another PR**
 
 **To this:**

3. Its free mentorship from the maintainer of the code bases who in most cases tend to be senior engineers. These guys are more willing to help you simply because you've shown initiative which is something very few people do. Remember, most juniors want a mentor but by contributing to open source, you get a mentor by default. No asking required. 

4. Open source contributions expose gaps in memory and will allow you identify which areas you are not as proficient in as much as you claim to be. Take an example, when i was working on my first Open source contribution, I struggled a lot with writing tests and making the distinction between whether to write feature or unit tests. While it wasn't my first time working with Rspec and Capybara(the testing framework duo of the project), i certainly hadn't used these in a production environment. When i identified this gap, i went down a rabbit hole to answer questions like How is a test built line by line?, What is the difference between "describe", "it" and "context" and when should each be used? How do i stub and mock? That was back in 2024. Fast forward to June 2025, and I'm way better at writing tests than I was when i started out. At some point, i even considered starting with the test first and then implementing a feature last, something old me would've considered only possible for experienced programmers. The thought that i would even consider this is very funny given that, TDD is the one thing that never made sense to me for a while. 

5. There is no downside to contributing to open source. Even in unfortunate cases where your PR is not approved and merged, the sheer process of working on the PR, meeting the standards of that code base, fixing tests that are failing on the PR, all these are learning experiences in of themselves are lessons which you may draw upon when working on future problems in other code bases. 

6. Your contributions will serve as experience when applying for potential jobs. I haven't done this in practice personally, but its not unheard of for someone to say that they landed their current developer position just because they contributed to open source. 

Lastly and most importantly, you're working on something bigger than you, something for the community. Something relied upon by other people and their projects. That in of itself is fulfilling. Its honestly one of the best ways you can give back. 

