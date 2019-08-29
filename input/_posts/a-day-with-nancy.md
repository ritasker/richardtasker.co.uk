layout: post
Id: 36767814-02f5-4934-ba41-300cb3125e82
Published: 2013-07-24
Title: A Day with Nancy
Lead: Nancy FX is a light weight web framework, which allows you to get up and running much quicker than some of the existing frameworks.
Author: Rich Tasker
permalink: /2013/07/24/a-day-with-nancy
Tags:
  - C#
  - NancyFx
  - MongoDB
---
I have been using MVC almost continuously for 4 years, and I think its great. While it is easy to get a simple project off the ground with most things out of the box, there is a lot of extra fluff that comes with ASP.Net projects. Time to introduce [Nancy Fx](http://nancyfx.org/).

I have been meaning to look at Nancy for probably about a year now, yesterday I was able to sit down for a few hours and play.

Since February there has been a regular meet-up in Leeds for devs, [Code and Coffee](http://www.agileyorkshire.org/codeandcoffee), every Wednesday a few devs meet in a coffee shop and code together. I have never been able to attend, due to working outside of Leeds centre. Yesterday I managed to, my goal was to get a very simple Create, Read, and Delete app running.

Nancy is a light weight web framework, based on the ruby framework Sinatra, hence the name. It is really good at creating web APIs, and uses a set of sensible conventions to get you up and running asap.

To get started I created a new Empty ASP.Net Web Application. Then from the package manager console installed the Nancy.Hosting.Aspnet package, which has a dependency on Nancy so both are installed.

`PM> Install-Package Nancy.Hosting.Aspnet`

With Nancy installed I needed to create my first module. 

    public class HelloModule : NancyModule
    {
        public HelloModule()
        {
            Get["/"] = parameters => "Hello Nancy";
        }
    }

Nancy handles its URL route patterns in the constructor of the module, you can also tell the module a base URL, more on that later. In the above example all I am doing is returning text to the browser.

![](/assets/images/posts/2013/07/helloNancy.png)

With the hello world example out of the way, I wanted to start writing data to a database. I wanted to try out a document database, again this choice over SQL was down to going for the light weight approach. And [Mongo](https://www.mongodb.com/) was his nameo.

`PM> Install-Package mongocsharpdriver`

To get Mongo working with I needed to tap into some of the extensibility of Nancy, by writing a custom bootstrapper. 

    public class CustomBootstrapper : DefaultNancyBootstrapper
    {
        protected override void ConfigureApplicationContainer(TinyIoCContainer container)
        {
            string mongoUser = ConfigurationManager.AppSettings["MongoUser"];
            string mongoPassword = ConfigurationManager.AppSettings["MongoPassword"];
            var connString = ConfigurationManager.AppSettings["MongoUrl"];
            connString = string.Format(connString, mongoUser, mongoPassword);
            var databaseName = connString.Split('/').Last();

            var client = new MongoClient(connString);
            var server = client.GetServer();

            var database = server.GetDatabase(databaseName);
            base.ConfigureApplicationContainer(container);

            if (!database.CollectionExists("Comments"))
                database.CreateCollection("Comments");

            container.Register<MongoServer>(server);
            container.Register<MongoDatabase>(database);
            container.Register<MongoCollection<Comment>>(database.GetCollection<Comment>("Comments"));
        }
    }

There is a bootstrapper which lies in the core of Nancy, which is in charge of configuring the different parts of Nancy. I wanted to override the IoC container so I could register my Mongo dependencies. Looking at the last example, I am retrieving a URL and a user name and password from config, this URL points to a Mongo DB on App Harbour. I am not going into detail on how to set this up here, for more info see this [link](http://support.mongohq.com/partners/appharbor.html).

From the Bootstrapper you may be able to tell that my domain object is a Comment.

The Comment has an Id and a Text property. The BosnId decorator is to tell Mongo to automatically generate a GUID for the ID of the object.

Following Create part of CRD I added the AddComment method to my module first to insert the data into the database.

    public class HomeModule : NancyModule
    {
        private MongoCollection<Comment> _comments;

        public HomeModule(MongoCollection<Comment> comments)
        {
            _comments = comments;

            Get["/"] = Index;
            Post["/AddComment"] = AddComment;
            Post["/DeleteComment"] = DeleteCommente;
        }

        private Negotiator Index(dynamic parameters)
        {
            var model = new
            {
                Title = "Hello Nancy!",
                Comments = _comments.FindAll().ToList()
            };

            return Negotiate
                .WithView("Index")
                .WithModel(model);
        }

        private Response DeleteComment(dynamic parameter)
        {
            var query = Query<Comment>.EQ(m => m.Id, (string)Request.Form.Id);
            _comments.Remove(query);            
            return Response.AsRedirect("/", RedirectResponse.RedirectType.Permanent);
        }

        private Response AddComment(dynamic parameter)
        { 
            if (!Request.Form.Comment.HasValue)
                return Response.Context.Response.StatusCode = HttpStatusCode.BadRequest;

            _comments.Save(new Comment { Text = Request.Form.Comment });

            return Response.AsRedirect("/", RedirectResponse.RedirectType.Permanent);
        }
    }

The code above checks that the Comment has a value, if is does then it saves the comment and redirects to the index route else throw a 500 server error. 

Reading a list of comments was easy, the code simply calls the FindAll method, converts it to a list with Linq and returns it to the view to be displayed.

In the Delete Comment method the Id is being passed back from a form in the view. The Id is used in a query to retrieve the comment from the database, which is then removed in a separate call to Mongo. 

All in all I think it has taken me longer to write this post than it did to get this example running. 
I have really enjoyed working with Nancy due to it being easy to work with, I can see advantages in exploring Nancy further. It is definitely something I would consider using for future projects.
