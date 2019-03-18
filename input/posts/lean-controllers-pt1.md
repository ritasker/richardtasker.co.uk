Published: 2016-08-15
Title: Skinny Fat, Lean Up Your Controllers - GET Requests
Lead: In this post I will show you how to remove data access code from your GET actions to Lean Up Your Controllers.
Author: richtasker
Tags:
  - ASP.Net
  - MVC
  - C#
  - Architecture
---
<blockquote class="twitter-tweet" data-lang="en" align="center"><p lang="en" dir="ltr">Announcing v2.0.0 of Human.exe <a href="https://t.co/nBzeG4xGwr">pic.twitter.com/nBzeG4xGwr</a></p>&mdash; Richard Tasker (@ritasker) <a href="https://twitter.com/ritasker/status/751099475368747009">July 7, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

This post is a little belated my wife and I have recently had our second child, Bethany May, and she has taken some time to get used too.

Posts in this series:

- [Introduction](https://richardtasker.co.uk/2016/06/09/lean-up-your-controllers-intro)

Applications can either read data or write/delete data. In HTTP terms these are  GET/POST/PUT/PATCH/DELETE requests. In this post I want to refactor our controller to make reading data from a database more efficient.

    [HttpGet]
    public ActionResult Edit(Guid id)
    {
        try
        {
            var conference =  _dbContext
                .Conferences
                .First(x => x.Id == id);
        }
        catch(InvalidOperationException ex)
        {
            return new HttpNotFoundResult(ex.Message);
        }

        var model = new EditConference
        {
            Id = conference.Id,
            Name = conference.Name,
            Description = conference.Description,
            StartDate = conference.StartDate,
            EndDate = conference.EndDate
        };
        
        return View(model);
    }

Here is an example of a GET request in our example application. Most GET actions follow the same pattern of taking in one or more parameters and using them in a query. Then converting the results into an object for that view and returning the view result.

1) Query data from a database (possibly multiple queries)
2) Compose the query results into a view model
3) Return a HTML view.

This code isn't terrible, it is quite easy to follow. It even returns a 404 result if the record can't be found. Only step 3 has to stay in the controller action everything can be abstracted away. 
In the above example the query is quite simple, but there is the potential to have a much more complex view that queries the database multiple times and composes the result into one view model.

Lets encapsulate the data access code into one repository class.

    public class ConferenceRepository : IConferenceRepository
    {
        private readonly IContext _context;

        public ConferenceRepository(IContext context)
        {
            _context = context;
        }

        public Conference FindById(Guid id)
        {
            try
            {
                return _context
                .Conferences
                .First(x => x.Id == id);
            }
            catch (InvalidOperationException ex)
            {
                throw new EntityNotFoundException(nameof(Conference), id.ToString(), ex);
            }
        }
    }

This benefits the application in a number of ways.

1) If the query changes, the code only has to be changed in one place.
2) Also if the ORM or database technology changes the controller doesn't have to.

Currently the repository is returning the applications `Conference` object. I could use libraries like [AutoMapper](http://automapper.org/) to automatically map the domain model to the view model. Or I could write the data to the database in a read optimised format and use the ORM to map the object. 
Assuming the data has already been written the repository class would stay pretty much the same it would return different object from the methods, it would be the EntityFramework DataContext that would change.

    public class ConferenceRepository : IConferenceRepository
    {
        private readonly IContext _context;

        public ConferenceRepository(IContext context)
        {
            _context = context;
        }

        public List<ConferenceSummary> FindAll(int top, int skip)
        {
            return _context
                .ConferenceSummaries
                .Skip(skip)
                .Take(top)
                .ToList();
        }

        public ConferenceDetail FindById(Guid id)
        {
            try
            {
                return _context
                .Conferences
                .First(x => x.Id == id);
            }
            catch (InvalidOperationException ex)
            {
                throw new EntityNotFoundException(nameof(Conference), id.ToString(), ex);
            }
        }
    }

I have included the `FindAll()` method in the repository to show how the `DataContext` has changed. An additional property has been introduced on the context for the index page. The `ConferenceSummary` class contains the Conference `Id` and `Name`, while the `ConferenceDetail` contains more detailed information.

I am really happy at this point, the `ConferenceController` is now considerably thinner and only has to deal with UI concerns.

    public class ConferencesController : Controller
    {
        private readonly IConferenceRepository _repository;

        public ConferencesController(IConferenceRepository repository)
        {
            _repository = repository;
        }

        [HttpGet]
        public ActionResult Index(int top = 10, int skip = 0)
        {
            var model = _repository.FindAll(top, skip);
            return View(model);
        }

        [HttpGet]
        public ActionResult Edit(Guid id)
        {
            try
            {
                var model = _repository.FindById(id);
                return View(model);
            }
            catch (EntityNotFoundException ex)
            {
                return new HttpNotFoundResult(ex.Message);
            }
        }

        [HttpGet]
        public ActionResult Details(Guid id)
        {
            try
            {
                var model = _repository.FindById(id);
                return View(model);
            }
            catch (EntityNotFoundException ex)
            {
                return new HttpNotFoundResult(ex.Message);
            }
        }
    }

_"But you are returning a domain object from the database to the view!"_
I consider a domain object to be an object that has business rules attached to it. Here I am returning a dat object which could be composed from multiple data sources.

In the next post we will look at writing data and how we can remove it from our controllers.

-- Rich
