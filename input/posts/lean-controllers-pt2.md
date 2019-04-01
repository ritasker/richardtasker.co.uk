Published: 2017-10-25
Title: Skinny Fat, Lean Up Your Controllers - POST/PUT/DELETE Requests
Lead: This post focuses on the writes in the system and introduces the concept of a command and handlers for mutating state, reducing the controller further.
Author: richtasker
permalink: /2017/10/25/lean-controllers-pt2
Tags:
  - ASP.Net
  - MVC
  - C#
  - Architecture
---
Posts in this series:

- [Introduction](https://richardtasker.co.uk/2016/06/09/lean-up-your-controllers-intro)
- [GET Requests](https://richardtasker.co.uk/2016/08/15/lean-controllers-pt1)

This series looks at how to refactor a simple ASP.Net MVC web application, to produce skinny controllers that do no more than return data or hand off to a different part of the application.

> **UPDATE:** 9-Nov-2017 I have removed the return value `TResult` from the code examples as I didn't need to return a value from the handler in this instance.

In the last [post](https://richardtasker.co.uk/2016/08/15/lean-controllers-pt1) I refactored the queries in our GET requests into a repository and created read optimised tables for the queries to read from.

In this post we will be focusing on the Edit action of the Conference Controller and introduce the concept of a command for mutating state, and a handler to execute the command.

In the simplest of cases most controller actions that handle POST requests will do something like validate the request, if the request is invalid the validation errors would be returned to the client. If the request is valid then a model is loaded from a data source or created, properties are set then the model is saved back from whence it came. The controller may then redirect to a page in the site.

    [HttpPost]
    public ActionResult Edit(EditConference model)
    {
        if (!ModelState.IsValid)
        {
            return View(model);
        }

        var conference = _repository.FindById(model.Id);

        conference.Name = model.Name;
        conference.Description = model.Description;
        conference.StartDate = model.StartDate.Value;
        conference.EndDate = model.EndDate.Value;

        _repository.Save(conference);

        return RedirectToAction("Details", new {model.Id});
    }

A command is something that changes the state of the system. Commands are declarative in nature and modifications to the state should be idempotent. For example imagine the cruise control in your car, you wouldn't have an `IncreaseSpeed` command which adds x mph to the current speed as resending this command would continually change the speed of the car. A `SetSpeed` command which sets the speed of the car to the sent value could be resent.

The result of the command should be easily inferred from its name. If the date of a conference has to be moved we will want to issue a `RescheduleConference` command. 

As a first step lets remove the `ModelState.IsValid` code from our actions. This can be done using an Action Filter.

    public class ValidatorActionFilter : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            if (filterContext.Controller.ViewData.ModelState.IsValid) return;

            filterContext.Result = new ViewResult
            {
                ViewName =$"~/Views{filterContext.RequestContext.HttpContext.Request.Path}.cshtml",
                ViewData = filterContext.Controller.ViewData
            };
        }
    }

It sets the requests result as a view result using the relevant view and the invalid view data.

Now the validation code has been removed, the main chunk of our action code is what the handlers will be doing. Below is an example of what I want a handler to look like.

    public class MyHandler
    {
        public void Handle(MyCommand command)
        {
            // Do stuff here
        }
    }

We can define the Handler and Command, like so.

    public interface ICommandHandler<TCommand> where TCommand : ICommand
    {
        public void Handle(TCommand command);
    }

    public interface ICommand{}

~~Using `TResult` allows a value to be returned to the controller from the handler, you may want to pass back a generated `Id` to redirect to a page or set a location header.~~

Using these interfaces our command handler will look like this.

    public class RescheduleConference : ICommandHandler<RescheduleConference>
    {
        public void Handle(RescheduleConference command)
        { 
            var conference = this.repository.FindById(command.Id);

            conference.StartDate = command.StartDate.Value;
            conference.EndDate = command.EndDate.Value;

            this.repository.Save(conference);
        }
    }

And our command

    public class RescheduleConference : ICommand
    {
        public Guid Id { get; set; }
        public DateTime? StartDate { get; set; }
        public DateTime? EndDate { get; set; }
    }

Now our controller action will look something like,

    [HttpPut]
    public ActionResult Reschedule(RescheduleConference command)
    {
        this.commandHandler.Handle(command);
        return RedirectToAction("Details", new {command.Id});
    }

Note I have made the action more declarative by renaming the method from a general `Edit` to `Reschedule`. If you have an publicly available API you may want to preserve the existing endpoint. You could do this by creating and calling multiple handlers.

In the next post we will look at updating the read model so everything is in sync and introducing some business logic to our entity.

-- Rich
