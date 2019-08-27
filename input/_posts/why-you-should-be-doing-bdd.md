layout: post
Id: aa857087-5dd1-4924-8ebc-6afe2dbf8ada
Published: 2014-04-22
Title: Why You Should Be Doing BDD
Lead: BDD is a bit of a buzz word at the moment. People seem to think its just a new way of writing tests. But it is not just about the tests...
Author: Rich Tasker
permalink: /2014/04/22/why-you-should-be-doing-bdd
Tags:  
  - BDD
  - Agile Methodologies
  - Public Speaking
---
This is a post to accompany my presentation. The slides can be found on my SpeakerDeck profile [here](https://speakerdeck.com/ritasker/why-use-bdd).

Behaviour Driven Development [BDD] is a bit of a buzz word at the moment, to understand why I am providing a small introduction to the subject and why it is a great process to use in writing software.

Originally coined around the mid 2000's by Dan North, BDD is an evolution of Test Driven Development [TDD]. Which also takes concepts from existing Agile practices and Domain Driven Design [DDD] to produce a process not just for testing software but for the whole software development cycle. BDD is great for teams who are new to testing or have recently developed software with Agile methodologies. 
BDD also helps teams workout what the next most important feature is, when a feature is finished, where to start developing and what to call objects in the system.

How does BDD do this? BDD takes everything we have learn't from TDD and adds to it. There are lots of blogs and books on the subject of TDD, but in essence a developer writes their test code before their production code. A developer would write a failing test then write just enough code to make that test pass. Then the developer can refactor the code and repeat the process until the feature they are working on is done. Hence the mantra Red, Green, Refactor.

BDD shows us that by writing our test names as sentences we can solve the common problems and understand,

- What to call our tests
- What to test and what not to test
- Why our tests fail

## What to Call Your Tests

    public class ClientDetailsValidatorTests
    {
        [Fact]
        public void Validate_Fails_MissingFirstName()
        {
            // test code
        }
    }

When I first starting doing Test Driven Development my tests looked something like the example above. It is not instantly clear what the production code should do, you need some understanding of the test code.
BDD tells us to write our test names as sentences describing the behaviour we are wanting to implement. Starting the test name with "Should" and following the sentence template "My class/method *Should* do something." helps focus the test. For example,

    public class WhenAskedToValidateSomeClientDetails
    {
        [Fact]
	public void ShouldFailWhenFirstNameIsMissing()
	{
	    // test code
	}
}

This is a lot more understandable to none technical people on the project.

## What to test and what not to test

There is only so much behaviour you can put into one short sentence. I have found that if you are finding it hard to write the test name or if the word *And* appears in your Should part of the test name and it is becoming complicated then this is a good sign you maybe testing to much or you have behaviour in the wrong place.

    public class WhenAskedToValidateSomeClientDetails
    {
        [Fact]
        public void ShouldFailAndSendAnEmailWhenFirstNameIsMissing()
        {
            // test code
        }
    }

Here our test is checking that when the first name property is missing on the client details object the validate method will fail and send an email. If this test fails we will have two things to check, whether the validate method passed or failed and if the email service was called or not.
But more importantly if you implement this test you will be breaking the Single Responsibility Principle, The Validation class should know nothing about email, we have behaviour in the wrong place.

## Understand why Tests Fail

As your code evolves over the course of the project tests will fail. They will fail for one of three things,

- A bug has been introduced
- Behaviour has moved
- Behaviour has been removed

To understand why the test is failing go back to the test read the test name to understand the expected behaviour. Read the code, has a bug been introduced? then fix it. Has a piece of behaviour moved, move the test and refactor it if needed. Has behaviour been removed, then delete the test.

As I said earlier BDD is not just about testing (although I do talk more about testing later), BDD can be used for the whole software development process. BDD can be used in the requirements gathering stage as well. By using some common other requirement gathering techniques and some of the concepts from DDD a Business Analyst can workout and prioroties the requirement of the system. DDD is quite a large subject in itself and would take a few posts to describe it. One of the main ideas behind DDD is that communication is key when modelling the system. To aid the discussions between the business and technical experts a common or ubiquitous language is produced. This ubiquitous language describes objects and processes in the real world and the same objects and processes in the system. The point of the discussion is to workout the requirements of the system. A great way to capture these requirements in through user stories.

## User Stories

A User Story is a short paragraph that describes a feature or requirement of the system. They can also be used to prioritise the feature. A common template can be seen below.

    As [a User]
    I want to [do something]
    So that [I can complete a task]
	
I like to use this version.

    In order to [complete a task]
    As [a User]
    I {Must/Should/Could/Wont} [do something]
	
Rearranging the paragraph likes this focuses the user on the task in hand, and what steps need to be done to complete it. I also use the [MoSCoW Priorities](http://en.wikipedia.org/wiki/MoSCoW_method) (Must/Should/Could/Wont) method to factor in the business value this feature would bring to the system. Here is a simple user story for an ATM from the customers point of view.

    In order to avoid waiting in line
    As a Bank Customer
    I should be able to withdraw cash from the ATM

For every story that is written there are many scenarios to consider, for instance in the ATM example.

- Is the ATM in service
- Has the ATM ran out of money
- Is the Customers Account in credit

BDD provides a way to capture these scenarios through another templating language called [Gerkin](https://github.com/cucumber/cucumber/wiki/Gherkin).

    Given some initial setup
    When an event occurs
    Then ensure some outcome

Looking at the ATM user story we can produce the following scenarios around the customers account being in and out of credit.

    Given the account is in credit
    And the dispenser contains cash
    When the customer requests cash
    Then ensure the account is debited
    And the cash is dispensed

    Given the account is not in credit
    And the dispenser contains cash
    When the customer requests cash
    Then ensure the account is not debited
    And the cash is not dispensed

Looking at these scenarios it can been seen that they both hang off the same `When the customer requests cash` event, but due to the initial setup the system behaves differently producing different outcomes.

These scenarios can be considered as acceptance criteria for a feature. If the system passes these acceptance criteria then the system is behaving correctly. But instead of handing the acceptance criteria back to a person to check if the system is behaving correctly BDD tells us that the acceptance criteria should be executable. Making the acceptance criteria into acceptance tests reduces the feedback time to the developer (is the system behaving as requested) and the client (as a progress report). This style of Given, When, Then testing is called xBehave.

A simple ATM example of both styles of test can be found on my [GitHub](https://github.com/ritasker/Pier8.BddExample) page.

In my opinion this is where BDD adds a lot of value to the development process. Writing user stories and acceptance criteria encourages communication with the client, the feature is as they expect. And making the acceptance criteria executable, makes everyone aware when a feature is complete and behaving correctly.

## Why you Should be doing BDD

Hopefully I have given you a good overview of BDD but if I still haven't convinced you here some points to take with you.

For developers,
 
- If you are new to testing BDD will help get into good habits fast.
- Your tests will be more focused, you will be able to see when behaviour is in the wrong place.
- And you will be able to understand quicker why a test is failing

For non technical members of the team,

- Some of the techniques can help prioritise the features
- BDD aids communication, as everyone will be talking the same ubiquitous language
- Reduces feedback time, when an acceptance test passes everyone can see when a feature is done or behaving correctly. Your customers can also see if a feature is not quite right.
