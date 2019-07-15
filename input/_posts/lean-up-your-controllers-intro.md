layout: post
Published: 2016-06-09
Title: Skinny Fat, Lean Up Your Controllers - Introduction
Lead: Fat controllers are bad. In this series I will show you how you can lean up your controllers by moving your code to a command based system.
Author: Rich Tasker
permalink: /2016/06/09/lean-up-your-controllers-intro
Tags:
  - ASP.Net
  - MVC
  - C#
---
Over the years I have worked on a number of projects that have had *"Fat Controllers"*. People are always talking about having fat controllers is bad and skinny controllers are the way to go. When applications grow, scale becomes an issue they be right. You don't want long running queries holding up your users and business logic repeated across your controller actions.

In this series I want to show you how you can cut the flab and lean up your controllers by refactoring and moving your code to a command based system. I will be demoing these techniques through a simple conference management application, which you can find on my [github](https://github.com/ritasker/ConfApp).


#### What's Wrong with Fat Controllers?

Let's start by defining what a fat controller is. Well a fat controller probably has hundreds of lines of code, with action methods, lots of action methods, each of them with massive chunks of hard to follow code. A fat controller will be doing too much and have lots of dependencies on services. Business logic and tight coupling to the domain has creeped into the controller.

All in all, this lends to complex, unmaintainable and hard to test code.


#### Wrapping up
IMO the worst thing you can have in your controllers is business logic, you should be able to swap out your MVC project for a <insert super cool new technology here> project with minimal effort and no loss of behaviour.

Over the posts in the series my goal is to,

- Eliminate business logic or data access code in our actions
- Make our actions behaviour declarative
- Simplify our code so it is maintainable for others
- Reduce and refocus our tests.

As we progress I would love to hear your thoughts, so leave a comment below.

-- Rich
