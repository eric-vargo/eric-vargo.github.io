---
layout: post
title: External REST API Guidelines
date:   2015-06-01 13:45:44
---
When creating a REST API it is important to remember to do what works best for the organization.  There are no true standards, only best practices and guidelines.  It is also important to take into consideration if there will be external consumers of the API.  External-facing APIs should be a bit more stringent in adhering to best-practices since there is basic set of expectations that most users of APIs have come to expect.  This will create an API that will feel familiar to users and avoid unnecessary confusion.  Given that, there is great variance among even the most popular and widely-used APIs so keep that in mind when meetigs devolve into ideological warfare; It probably doesn't matter, as long as the API is clean, logical, and well-documented.

This post is merely a compilation of concepts to keep in mind when creating REST APIs. Some of these should be considered all of the time, while others should only be used when they support the needs of the organization.  This is not a definitive or comprehensive list.  It is, as the title suggests, a set of guidelines that I have grown to like during my travels as a software engineer and API designer. Also of note is that I do not indend this to be an "original" document.  It is a collection of excerpts I have jotted down over the years, taken from various sources and embelished upon. I am not misrepresenting this as a whole, as my own. It's probably about 60-70% other people's work. This is a collection of thoughts that together, make up the guidelines that I think are the most importent to consider when desiging a REST API.

### Main Goals to Consider When Designing a REST API
- Keep the client's perspective always at the forefront
- Hypermedia links (HATEOAS)
    - Canonical URL for navigation
    - Link traversal
    - I'm still not sold that there is great value from a programmatic perspective, but I have found link traversal very useful when manually browsing through a public API, such as Github. Think of using these links, at the very least, as an extension of documentation.
- Content types to support
    - content is negotiated through headers `Accept` and `Content-type`
    - use JSON...why even bother with XML?
        - If you need other content types, knock yourself out. Document it!
- Use as much of emerging standards as possible
    - Although not necessary, this will make some managers happier about their endeavors
    - [HAL](http://stateless.co/hal_specification.html)
    - [Collection+JSON](http://amundsen.com/media-types/collection/examples/)
        - Spring did this out of the box for their HATEOAS code
- Model your API after other APIs from prominent tech companies
    - This is also optional, but can help when selling a potential project to certain managerial types.
    - [Paypal](https://developer.paypal.com/docs/api/)
    - [Github](https://developer.github.com/v3/)
    - Salesforce

### REST API URLs
REST URI's should be thought of as a "path to a resource" or a "noun" in the system. The URI should provide a way to drill down your data to get to any given resource or set of like resources.  

This is pretty much the only set-in-stone standard for a REST API that I would say has universal acceptance. Deviations from this basic tenet should only arise when your resources have functionality that goes beyond HTTP verbs, such as a "submit payment" or "calculate" function.  (more on that later)

#### HTTP Verbs
_OPTIONS_
Requests should return a list of links available around a given resource. This could include whatever the organization would like here, be it only GET resource links or POST, PUT, DELETE, etc. If you decided against returning hypermedia links in your responses, an OPTIONS request probably isn't neccessary. One could say it is...optional.

_HEAD_
Functionally similar to a GET request, but without the body returned. Useful when used to obtain the ETag of a resource, or returning content types supported by the API. Supporting a HEAD request is also not necessary, but can be useful if you support many custom content types, for example.

_GET_
GET requests should be in the form: `/path/{id}/to/{id}/resource/{id}`
For the case where you have other ways of identifying your data, such as git repositories, you could do something like: `/repos/:owner/:repo`
Where in the Github case, you don't have the resource type followed by an id, you have a customer-specific way of drilling into your data.

All GET requests should return an ETag. This ETag ensures against a concurrent modification problem.
A client can check for an updated copy of a resource by giving the `If-None-Match header` along with the request and sending the ETag value.  This will return the resource only if the version has changed since the client's ETag. If the resource hasn't changed, a 304 response code should be returned.

Specifying a range of resources to return on a GET request for pagination can be done in a few ways:

- Using the HTTP "standard" Headers to help describe the resources
    - Servers should provide an `Accept-Ranges: resource` header to indicate to a client that they support resource-based range queries.
    - Client could then send header to paginate: `Range: resources=100-199`
- URL parameters: `?limit=10&offest=1` or `?page=1&pageSize=50`

There is much debate online about the "proper" way to perform pagination in a REST API. I think it really doesn't matter. For me, using parameters is simple and works fine.

If you have rather large resources in your application, supporting partial selection of a resource might make sense. This allows clients to select which properties of the resource to include in the response in order to limit the amount of data sent over the wire.
ex: `/kennel/123/dog/1234?fields=name,color,location`

_POST_
It is generally accepted that a post will only create a resource.  The full data of the resource is expected in the body.  Whether that includes complete nested data is up to the organization, i.e. you may require subsequent API calls to update nested data because it doesn't logically fit into one transaction. I have seen APIs where a POST request will also update data, but I think that violates the basic HTTP verb standard.  Keep POSTS strictly for create operations and have update operations sent via a...

_PUT_ and _PATCH_
The PUT request should be viewed as updating a full resource, i.e. a complete replacement. In the past, a PUT request has also been used for selective uodates. It is generally assumed that selected updates will be handled by a PATCH request going forward. PATCH is not yet the standard for selective updates, but is on it's way, so start using it now and you'll likely avoid confusion in the future.

More on selective updates...A PATCH request will only update what it is given, i.e. it will be a set of deltas to apply to the data. It can be in a few forms:

- nested
    - all objects are nested in the hierarchy
    - works better with smaller data structures and updates applied to most of the resource
- mapped
    - a key:value way of updating complex, nested objects
    - `{“person.address.state.code“: “MA”}`
    - works better with larger data structures and fewer updates per object

_TODO: Updating Collections:_ What happens when return codes should be different for each member of a collection?

_Other Verbs_
If you have functionality in your application that falls outside the standard HTTP verbs, such as a "submit payment" or "calculate" function, it is probably best to use a POST request. I suggest POST because I think of this as "creating" a new instance of the function.  You could decide to use PUT.  It doesn't matter as long as it is well-documented.

What to do on a “submit” example?
`/path/{id}/to/{id}/resource/{id}/submit`

A URL mapping such as this allows for cleaner code on the back-end when using a framework like Spring MVC. (That matters when designing an API! If you have a seemingly "6 or one half" choice to make, choose the option that your tool better supports.)

### Status Codes

#### Server Success Codes
200 - OK

201 – Created
Server has successfully created a new resource as a result of a POST request.  The new resource's location should be returned in the Location header.

202 – Accepted
Server has accepted the request, but it is not yet complete.  This status is useful for long-running tasks that may need to contact 3rd party resources, for example.  The response should contain a link pointing to a status monitor. Once the long-running create process has finished, the response from the status monitor should include the same headers and response body had the request been fulfilled synchronously.

#### Client Error Status Codes
If a client receives a 4** status code, that means the onus is on the client to fix a subsequent request.

400 – Bad request
The request could not be understood by the server due to malformed syntax. The client should not repeat the request without modification.

401 – Unauthorized
The response could include a `WWW-Authenticate` header in order to convey to the client what authentication schemes are supported by the server. If the request already included Authorization credentials, then the 401 response indicates that authorization has been refused for those credentials.

403 – Forbidden
Server has understood the request, but refuses to honor it. Client should not repeat the request without modification.

404 – Not Found
The server was unable to find the URI resource.

405 – Method Not Allowed
The method specified is not allowed for the resource identified by the request URI. The response should include an `Allow` header containing a list of valid methods for the requested resource.

406 – Not Acceptable
The server is unable to match the client's `Accept` header. Depending on your API, it could be more reasonable to default to JSON. As always, it depends on the goals of the organization.  

#### Server Error Status Codes
500 – Internal Server Error
This is an error from which the server cannot recover.  In theory, the client can do nothing to the request that will alter the result.  This is pretty much a catch-all for any server issue, such as database problems. All the client can do is wait and try again.

### Versioning a REST API

#### Versioning URLs
We can't expect clients to upgrade every time we make a change. Backward compatibility is paramount for any REST API (unless, of course, you are Facebook). It's a good idea to force clients to specify the version in all requests.

In versioning a REST API, there are generally a few ways to do it:
- put the version somewhere in the URL
- put the version in an HTTP header
- put the version in a parameter, such as ?v=1.0

It's easy to add another “V2” URL to have another version of the APIs, but I believe this to be more problematic than the header solution for one key reason; This creates a complete set of URLs for a particular version when you might only be changing a small percentage of them. Granted, this may be desirable from an organizational standpoint, but I think the better approach is to use the HTTP header solution.  I think having a verion in the URL forces a client to write a bit more code to maintain URLs through string concatenation. It's cleaner to set the header to a value.

Twilio had a different approach to this that I thought was clever. They force a user to send a date along with the request, and they will use the version of the API in place for that date and time.  So, a client will use the date they developed against the API. This prevents the client from having to explicitly know what version they are coding against.

On the server side, I believe code is cleaner when the version is a variable that is easy to access.  Tools such as Urlrewrite can easily route requests based on the version in a header, or url parameter for that matter.  It's a bit less clean to have to pattern match on a string.  Granted all of the reasons here for using an HTTP header are somewhat trivial, but it is cleaner and that is almost always my preference. It's too easy to write ugly code!

The tools used on the server side to process your REST APIs also should factor into the versioning decision. Ultimately, since it does not matter where the version is set, if your server side code can look a little cleaner by using one method compared to another, then let that guide your decision.

#### Versioning Domain Objects
You can avoid this type of granularity by tying all domain data versioning to URLs, but if you must have this level of granularity, you can determine the data version by creating custom media types.  The server can set what it produces, such as `application/json` or `application/com.yourcompany.person.v1+json`
You will then have a method that returns an object in your code called PersonV1. V2 returns PersonV2, etc...

### HATEOAS (one pronounciation: Hey-toes)
“Hypermedia As The Engine Of Application State”

The client doesn't have built-in knowledge of how to navigate and manipulate the model. The server will provide this information dynamically to the user implemented by using media types and link relations. Documentation should get clients started, with a canonical URL that supports an OPTIONS request operation. The service itself will guide after that.

#### Media Types
Client describes what it wants with the `Accept` header, and during a POST and PUT the “Content-Type” header. Server describes what it is sending with “Content-Type” header. If/When you decide to add strict adherence to a standard such as HAL, it will be easy for clients to request it using : application/hal+json, application/hal+xml

#### Link relations
The server describes these relations as part of its payload.  A link has 2 main parts: rel, href. You could also add the operation as some standards are advocating.  Rel values are static variable names that a client can recognize and rely on in order to obtain URLs to resources within your API.
`{“rel”: “doors”, “href”: “http://...”}`

Using HATEOAS means clients do not have to change their code if URLs change, assuming they are getting their URLs dynamically using the links we give them in the API.  This needs to be documented; Clients can then choose to rely on your API to traverse it.  As I stated earlier, in its basic form, I'm not sold on the usefulness of HATEOAS from a programmatic standpoint.  I think it's more useful as a documentation feature of your API.

Using HATEOAS also opens up more advanced possibilities to implement things like API chaining in order to make multiple API calls with one request.  This involves giving a chain piece of json with the “rel” values from the HATEOS links:

	chain :{
		“stuff_update” : {data to send, could be something like return val of previous chain},
		“send_email” : {people to notify you did something}
	}


### Final Thoughts
There are no standards, so whatever is decided, it needs to be well-documented on how it works and what the API expects.
If you deviate from any of the golden rules, make sure it becomes easier to use and you have removed complexity, not added to it.

_References_
A great presentation [by Ben Hale from a Spring One Conference](https://youtu.be/fSFh9UCBp5s)
