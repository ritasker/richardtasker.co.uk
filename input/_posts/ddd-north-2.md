layout: post
Published: 2012-10-23
Title: DDD North 2
Lead: DDD North is a free event for developers and organised by developers. These are the sessions I attended.
Author: Rich Tasker
permalink: /2012/10/23/ddd-north-2
Tags:
  - DDD North
  - Public Speaking
---
Last Saturday (13/10/2012) I attended the second DDD North event at the University of Bradford (where I graduated in 2007). It was a great day with an early start and a late finish.

Throughout the day I was able to met many developers old and new and chat about any cool projects and technologies they have been working on. In the evening I got another opportunity to meet new people, but this time the with beer and curry, at the Geek Dinner. I even manage to talk to Jerry Karlin inverter of the [RAM drive](https://en.wikipedia.org/wiki/RAM_drive#History_and_operating_system_specifics). 

### The Sessions


### Spatial SQL - [Peter Shaw](http://twitter.com/shawty_ds)

The session could not have come at a better time as it was only very recently that I was introduced to spatial SQL. My initial intro to spatial SQL came about due to the requirements of a new project at work to report all locations from our system within a 30 mile radius.

The presentation started with when to start using the spatial features, some common usages e.g. capturing location details from smart phones. The section on the OGC standard didn't interest too much, but it was good to know that most of the tools available follow the standard.

The session recommends PostgreSQL over MSSQL with the statement, MSSQL is the "Most compatible, incompatible implementation" of Spatial SQL. And MS adhering to 75% of the standard in their own way. Which was a bit of a blow as I currently on the .net stack, but not to worry as we can use SQLite DLL, as it implements the spatial features well.

The session finished with a few examples of some of the common queries used. A great introduction and hopefully some good discussions back at work.

### Riting Roslyn Refactorings - [Guy Smith-Ferrier](http://twitter.com/GuySmithFerrier)

Roslyn is a library that can be included into a project and used to read, remove and re-factor code. I have never seen any sort of explanation of Roslyn before so was looking forward to this.

Guy started by explaining the steps involved in compiling C# code, then briefly explained some of the main features of Roslyn. Guy had a great demo of writing a re-factoring tool similar to FXCop and StyleCop.

I would like to see more of Roslyn, especially seeing it write, compile then execute code on the fly.

### BDD, Look Ma No Frameworks - [Gemma Cameron](http://twitter.com/ruby_gem/)

I was torn between this session and Gary Short's on the Raspberry Pi, so much so I [tweeted](http://twitter.com/ritasker/status/256666304004636672) about it. I was persuaded with a promise of seeing a girl mounted on a unicorn, and that promise was [fulfilled](http://lancsrubygem.files.wordpress.com/2011/06/photo-e1308059032987.jpg?w=350&h=425), but not how I'd pictured it.

This was not my favourite talk of the day, which was not to do with Gemma or the content of the presentation but more to do with the lack of audience participation. The session did reinforce my existing knowledge of BDD and highlighted a different way of writing tests.

### WebSockets & SignalR, Building the Real-Time Web - [Chris Alcock](http://twitter.com/calcock)

After lunch and the [Grok Talks] http://wiki.developerdeveloperdeveloper.com/Default.aspx?Page=DDDNorth2012-Grok-Talks&NS=&AspxAutoDetectCookieSupport=1), was the session I was looking forward to the most. Coming from a robotics background seeing code react to events in real time really excites me.

Chris had made a simple drawing app using two canvas elements and the websockets protocol to transmit pixel data from one of the canvas elements to a web server. The server executed some code which returned a mirror image version of the pixel back to the web page where it was displayed on the other canvas element.

I really enjoyed the session (even if I was feeling sleep as it was the first session after lunch and there was a distinct lack of coffee) and the fact that data can be pushed from a server will bring some interesting new innovations to the web. 

### Event Driven Architectures - [Ian Cooper](http://twitter.com/ICooper)

This was my favourite session of the day. I thought the presentation would be along the lines of [Aspect Oriented Programming](https://www.google.co.uk/search?q=Aspect+Oriented+Programming), but I was wrong.

The session talked about [Service Oriented Programming](https://www.google.co.uk/search?q=Service+Oriented+Programming), where you split large monolithic applications in to smaller applications, or services, which interact with each other through events.

To explain the different aspects of the subject Ian used a hotel as an analogy.

The hotel had various departments (booking, banking and cleaning) each providing there own services. On the hotel website customers could book and amend their hotel reservations.

When a booking is made a new-booking event would be raised by the booking service this would cause a reaction from the banking service which would try to charge the customer. If successful the banking service would raise a success event likewise would raise a failure event if unsuccessful. The cleaning service would work in the same way and would react to the booking service by adding the event of cleaning that room into the cleaning schedule. Once the room is clean a new event would be raised by the cleaning service which the next service in the chain would pick up on and react to.

This was really interesting talk and coupled with some of the real time technologies from the previous session would make a pretty cool project.
