Published: 2015-02-04
Title: API Versioning in Web API
Lead: There are many methods and opinions on how to version an API. This post shows how to version a .Net Web API using accept headers.
Author: richtasker
permalink: /2015/02/04/api-versioning-in-web-api
Featured: true
Tags:
  - C#
  - ASP.Net
  - Web API
---
I haven't blogged for a while, I think I am still adjusting to fatherhood, but here is a small post to hopefully kickstart the habit again.

I am currently working in the API team of a company. We are building an API which, at the moment, will only be consumed by our own front end teams. This has meant releases of the API have been managed and coordinated between the teams therefore versioning has not been necessary.
However this did not last, due to sprint alignments and future commitments we needed to implement a breaking change to the API. Being a fairly greenfield project adding versioning wont be a hard job, but which method of versioning do we want to implement and who don't we want to piss off.

## The Different Versioning Methods

There are three common schools of thought on versioning,

* __Version in the URL__ This is probably one of the easist to implement and therefore the most common. All you have to do is put the version in
the URL or as a query string parameter. `/api/v2/myresourse` or `/api/myresourse?version=2`
* __Version in a Custom Header__ Without modifying the URL the version is put in a custom header like `api-version: 2`. While this method has nice 
RESTful URLs, the HTTP spec already has way for the consumer to ask how they want a resource to be represented, via the Accept header.
* __Version in the Accept Header__ Like the previous method this is more complicated to implement than versioning in the URL and also you now have 
to build a request up rather than having a URL that you can GET, POST, PUT, etc. to. This is my personal preference and will be what I implement below.

## Versioning the API with the Accept Header

What I would like to have at the end of this work is something like the example below.
    
    [HttpGet, RouteVersion(\"api/MyResourse\", 1)]
    public HttpResponseMessage Get(int id)
    {
        // Some code
    }
    
    [HttpGet, RouteVersion(\"api/MyResourse\", 2)]
    public HttpResponseMessage GetV2(int id)
    {
        // Some different code
    }
    
When Web.API 2.1 was released at the beginning of last year there were improvements to the Attribute routing. Web.API now supports constraints which will allow us to select a route based on header values. Using the `RouteFactoryAttribute` class we can create a custom route attribute and populate a list of `IHttpRouteConstraint` constraints. 

    public class VersionedRouteAttribute : RouteFactoryAttribute
    {
        public VersionedRouteAttribute(string template, int allowedVersion) : base(template)
        {
            _allowedVersion = allowedVersion;
        }
        
        public override IDictionary<string, object> Constraints
        {
            get 
            { 
                return new HttpRouteValueDictionary
                {
                    { \"version\", new VersionConstraint(_allowedVersion) }
                };
            }
        }
        
        private int _allowedVersion;
    }
    
The `IHttpRouteConstraint` has one method `Match`. This is where custom code is needed to workout if the incoming route matches the constraints.

    public class VersionConstraint : IHttpRouteConstraint
    {
        private const int DefaultVersion = 1;
        
        public VersionConstraint(int allowedVersion)
        {
            _allowedVersion = allowedVersion;
        }
        
        public bool Match(HttpRequestMessage request, IHttpRoute route, string parameterName, IDictionary<string, object> values, HttpRouteDirection routeDirection)
        {
            if (routeDirection == HttpRouteDirection.UriResolution)
            {
                int version = GetVersionFromHeader(request, parameterName) ?? DefaultVersion;
                
                return (version == _allowedVersion);
            }
            
            return true;
        }
        
        private int? GetVersionFromHeader(HttpRequestMessage request, string parameterName)
        {
            var acceptHeader = request.Headers.Accept;
            foreach (var mime in acceptHeader)
            {
                if (mime.MediaType == \"application/json\")
                {
                    NameValueHeaderValue version = mime
                        .Parameters
                        .FirstOrDefault(v => v.Name.Equals(parameterName, StringComparison.OrdinalIgnoreCase));
                    
                    int parsedVersion;
                    return int.TryParse(version.Value, out parsedVersion) 
                        ? parsedVersion
                        : (int?)null;
                }
            }
            return null;
        }
        
        private int _allowedVersion;
    }
    
From the example above, when a new `VersionConstraint` is created an allowed version value is passed in. When the `Match` method is called the code first checks if this is a URL resolution request, if it is we can try to get the version number from the Accept header. The `GetVersionFromHeader` loops through the header value collection in the Accept header and finds the first parameter where the name matches the parameter name passed in.
This is `\"version\"` and is set in the `VersionedRouteAttribute` when we add the constraint to the `HttpRouteValueDictionary`. If a version parameter is found the code will try and parse the value to an integer and return the number else return null.
 
Back in the `Match` method if a version number was not returned by the `GetVersionFromHeader` method the code will use a default value.
Finally if the header version equals the allowed version passed in at the start, we have a match and the app can execute the route action.
 
Using the accept headers is my choice over the other methods and I hope this implementation is useful for you, just don't shoot this messenger.
             
