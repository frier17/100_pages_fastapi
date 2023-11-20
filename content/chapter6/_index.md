+++
title = 'Middleware in FastAPI'
date = 2023-11-05T22:01:36+01:00
weight = 8
draft = true
+++

Middleware is a term used across different web frameworks and cloud base systems. Middleware can be seen as any software
or service that allows other systems to communicate of manage data. Middleware can also be seen as a service that acts
as an intermediary between two systems. The Starlette framework allows the use of Asynchronous Server Gateway Interface 
(ASGI) Middleware which can be used to process requests or responses before passing control to other processes. 
FastAPI also supports the use of middleware both bespoke and standard middleware included in the Starlette framework.

## Standard middleware

Middleware affects the behaviour of the entire web application. Once applied, all path operations will be modified by
the middleware. Using this approach, developers can modify all requests or all responses to meet a given need. As an
example, the `CORSMiddleware` allows application to set the right header parameters to enable cross-origin requests by
browsers. Every Starlette application has `ServerErrorMiddleware` and `ExceptionMiddleware`. Any other added middleware
is processed in the order in which they were registered with the application. FastAPI extends the Starlette framework
hence it supports these middlewares (`ServerErrorMiddleware` and `ExceptionMiddleware`). Like middleware registered in
Starlette application, FastAPI middleware work with every HTTP request before passing it to a path operation and work
with every response before returning an actual response. Middleware defined in Starlette can work with FastAPI
application seamlessly.

There are several middleware applications defined for Starlette. The summary table (Table 1) below shows standard
middleware for the `starlette` package.

{{<table title="Table 1" style="table-hover" caption="Standard middleware applications in starlette package" >}}

| Middleware                | Description                                                                                                                                                          |
|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CORSMiddleware`          | Manage response of application to ensure appropriate CORS headers are added to allow cross-origin requests.                                                          |
| `SessionMiddleware`       | Adds a signed cookie session that can be accessed as `requests.session` dictionary. Entries in the session are readable and cannot be modified.                        |
| `HTTPSRedirectMiddleware` | Ensures all incoming requests are directed to HTTPS or WSS. Requests using unsecured schemes (for example, HTTP) are redirected to HTTPS or WSS                      |
| `TrustedHostMiddleware`   | Ensures correct Host property is set for all incoming requests. A set of host names can be specified using allowed hosts parameter when registering this middleware. |
| `GZipMiddleware`          | Manages responses to requests needing the gzip compression or with gzip specified in Accept-Encoding header.                                                         |
| `BaseHTTPMiddleware`      | The base class for defining custom ASGI middleware classes.                                                                                                          |

{{</table>}}

## Custom middleware

Users can also write their own middleware to meet their needs. Let us suppose in our P2P system, we do not wish to save
client information like IP address or OS as plain text for security reasons. We will want to encrypt certain information
and then allow only processes or users with certain access view this information. One approach to use is to define
a `PrivacyMiddleware` which will scramble or encrypt our client information before this data can be saved to the
database. With this class, we can manage what aspects of our client is visible or hidden.

The `PrivacyMiddleware` is design to implement principles from the
work [Privacy by Design](https://gdpr-info.eu/issues/privacy-by-design/) by Ann Cavoukian. Details of privacy schemes or
techniques are beyond the scope of this book. In this book, we will use middleware to address some user concern and
encourage the reader to explore best practises around data security and user privacy. Our design approach is described
below.

A user request is searched for a valid token. If a token exists, then the user is assumed registered, and the
authentication process can verify the registered user. If there is no token, then the request is from an unregistered
user. For this case, the user details like IP address, OS, location, and other header data are copied to a temporal
variable. If the request has a body variable named privacy, or header with privacy, then a dictionary of information
that should be passed through the privacy function is retrieved. This dictionary will specify the attribute to be
encrypted as keys and the values are the token (nonce) for generating a symmetric key needed to encrypt or decrypt this
field. If a key has no corresponding value or its value is set to None, then a key is generated. Once the privacy
dictionary has been processed, the system privacy registry is updated. This registry will be a lookup object from which
all future requests will be processed for privacy per user for a given session. So future requests will be passed to
privacy function that will call the privacy registry for fields to encrypt. If these fields exist in the body or header
of the requests, they are copied, and then encrypted. The encrypted values are then used to replace the original request
values and then the request is passed on to the path operation or next process.

Privacy handling middleware to encrypt or parse user data.
{{<figure src="privacy.png" title="Figure 1" caption="Sample screenshot of PrivacyMiddleware encrypting user data as a measure of improving user's privacy" >}}


#### 📚 Further Reading - Practical privacy approach

> Privacy management is a complex subject that goes beyond just technical components such as cookies management or online
tracking. There are legal, technology, ethical and other aspects when dealing with privacy. A detailed discussion on
privacy is beyond the scope of this book, however, it is important for designers to plan for privacy from inception of a
system that will interface with users and possibly use data provided by a human user. A gold standard for managing the
technical aspect of privacy is Differential Privacy.
> + 📖 [Practical privacy by Katherine Jarmul](https://www.oreilly.com/library/view/practical-data-privacy/9781098129453/)
> + 📖 [Introduction to Differential Privacy](https://desfontain.es/privacy/differential-privacy-awesomeness.html)
> + 📖 [More detailed explanation of Differential Privacy](https://desfontain.es/privacy/differential-privacy-in-more-detail.html)
> + 📖 [OIPC - Privacy by Design Resources](https://iapp.org/resources/article/oipc-privacy-by-design-resources/)
> + 📖 [7 Principles of privacy by design](https://www.onetrust.com/blog/principles-of-privacy-by-design/)
