+++
title = 'Identifying Routes'
date = 2023-11-05T21:48:06+01:00
weight = 5
draft = true
+++


Before writing any line of code it is important to state or know exactly what the system will do. Defining what the
system does in a concise statement not longer than two or three sentences is very important to have clarity of purpose.
Programming is a complex psychological act and being precise is a bedrock in building the right system. Added to
building the right system is building the system right. This can be achieved by explaining how the system works. Both
knowing what the system does and how the system does its tasks are important. Taking our project, mWallet as an example.
We can derive the tests cases from knowing "what the system does" and we can derive our application code base from
knowing "how the system works".

## What the system does

> mWallet allows users to post their interest in buying or selling cryptocurrencies and allows registered agents (other
> users) to fill these orders without the requirement for a direct person-to-person exchange or conversation.

The statement above gives a direction on what vital services will be available in the mWallet project. The services
identified from the purpose statement are:

1. Users can post order to buy or sell cryptocurrency
2. Users can satisfy published orders

Let's name these key functions of our system the functional requirements. In effect, if our system cannot perform these
tasks we can say we failed to meet our objectives. Having identified these requirements, we can define test cases that
will check if our system can post an order, buy an asset through posted order, sell an asset through posted order, debit
the buyer accordingly while equally crediting the seller of the asset.

Tests can be written as a function prefixed with the word `test_` or as methods of a class which is often prefixed with
the word `test`. It is good practice to have all tests in a package named tests. We will be using `pytest` for the
project and it can run both tests written as classes that extend `unittest.TestCase` or functions prefixed with `test_`.

{{<code lang="py3">}}

from unittest import TestCase


class TestTransaction(TestCase):

    def setUp(self) -> None:
        ...

    def tearDown(self) -> None:
        ...

    def test_place_order(self) -> None:
        ...

    def test_satisfy_order(self) -> None:
        ...

    def test_transaction_history(self) -> None:
        ...

    def test_current_orders(self) -> None:
        ...

{{</code>}}

The test above can be written as a collection of functions as shown blow.

{{<code lang="py3">}}

def test_place_order(order: dict) -> None:
    ...


def test_satisfy_order(transaction_id, order, account, wallet, escrow, ) -> None:
    ...


def test_transaction_history() -> None:
    ...


def test_current_orders() -> None:
    ...


{{</code>}}

Our identified tests will also define the various URLs or Application Programming interfaces (API) endpoints of the
system. If our system is interactive, it is expected to give information or some sort of data to the user. Let’s call
any valid information given by the system a resource. A resource can be a digital file to download, a record from a
database or any web page we wish to share. This resource can be named and accessed via its name. Let us attempt to use
intuitive names to identify resources for our API endpoints.

We will name the following resources which can have associated functions for handling HTTP requests from our users.

+ `Order Resource`
+ `User Resource`
+ `Asset Resource`
+ `Cryptocurrency Wallet Resource`

🔍
> Details regarding designs of RESTful web applications are beyond this book. A good place to get more information on how
to design a RESTful application is the [swagger documentation](https://swagger.io/resources/articles/best-practices-in-api-design/).

We now have a collection of tests, a set of resources, but we need to check if these can help us reach our system
requirements. To meet our requirements, certain other questions need to be asked. These questions should provide answers
to how the system works. We will progress on this in the next section.

## How the system works

mWallet is a P2P system designed to satisfy orders both online and offline. To achieve this, orders are posted to users
who can then satisfy them as agents. Orders can be fully or partially satisfied, and payments remitted appropriately.
Orders are published to the pool of agents via email and in-app messages. Offline transactions are like real time
transactions but kept open for a longer period.

Offline transactions require an authorization token or passcode from an agent before adding them to the order.
Transaction keys enforce password-less transactions and limit access to wallets and order system for unauthorized or
unregistered agents. Escrow accounts and managed wallets protect users from exposing sensitive banking or wallet
information. This approach is used both by the real time and offline payment system. Details on the system encryption,
random function for selecting agents from a pool, and generating escrow accounts for payments are beyond the scope of
this book. With these guidelines on how the system should work, let us consider how the identified resources can address
the system behaviour.

+ The Order resource can manage both online and offline requests to buy or sell cryptocurrency.
+ The User resource can manage registration of new system users and agents, the user associated transaction keys, the
  user associated bank accounts and user access control.
+ The Asset resource will manage a list of cryptocurrencies that can be bought or sold on the platform.
+ The Wallet resource will manage escrow wallets for receiving digital assets.

We will now attempt to match a resource to a set of URLs that will enable users to interact with the system.

+ Order
    - Add order
    - View order
    - Edit order
    - Delete order
    - Add order to pool
    - Satisfy order
    - Debit buying party
    - Credit selling party
    - List transaction history
    - Assign order to an agent or agents to satisfy
+ User
    - Add user (new user registration)
    - View user
    - Edit user
    - Delete user
    - Assign transaction key to user
+ Asset
    - List available digital assets
    - View exchange rate of digital asset to fiat
+ Wallet
    - Generate address for user
    - Assign cryptocurrency address or wallet to user
    - List blockchain transaction history for address
    - View transaction detail of an address

*The list excludes cryptographic key management, signals or event management, and utility functions used in the
project.*

#### ℹ️ Tips and Tricks - Protecting the user resource

> An alternative design of resource may include a `Transaction` resource, `Bank Account` resource, a `User Key`
> resource, and a `User Preference` resource. These will enable users to view transactions associated with their orders
> and manage various items associated to their profile. In our design approach for this book, user is treated as a special
> resource which is only added upon sign up and cannot be viewed, edited, deleted, or updated like other resources.
> Special functions and security checks are used to manage the user resource. This approach can also be adopted in a
> real-life project to reduce possible cases of updated or deleting a user’s record via unapproved admin users.

The summary above can be modified as we iterate through the system and get feedback from our target users. However, for
our example, we will keep things simple – possibly very simple to allow the reader to explore alternatives and expand on
the project on their own. From the above list, we can generate a set of URLs. We will save this in a `route.yml` file
which will be used to dynamically load API routes and permissions later in the project.

## A classic way of defining routes in FasAPI

A Uniform Resource Locator (URL), a subset of Uniform Resource Identifier (URI), is an identifier for accessing a
resource that is on the Internet and specifies what protocol to use in accessing a resource. A resource could be a web
page or some other file. For the web application in view, URLs will be used to send or receive information from the
system. There are key parts that make up a URL. Some key components of a URL include:

__The protocol or scheme__: This will be HTTP/HTTPS for RESTful applications. The protocol determines how computers
communicate. There are several standards like File Transfer Protocol (FTP) for file transfer, Hypertext Transfer
Protocol (HTTP) for web communication etc.

__Host name or domain name__: A resource on the internet can be accessed via its unique IP address. However, it is
easier to remember readable names than a set of numbers separated by dots `.`. Domain names provide a human readable
name that is mapped to an IP address. Example of a domain name is `www.yahoo.com`.

__Port number__: In some cases, a URL may specify a port number by which requests to the server are listened for.
Default port for HTTP servers is 80. This port number does not need to be specified in URLs. In many cases, other port
numbers may be used, example `8080` or `8000`.

__Path__: The path refers to location on a web server.

__Query__. In some cases, extra information may be passed to the web server by using textual fragments. The query
component is used to pass data to the server as a key-value pair after a question mark (?). Multiple query parameters
can be separated using ampersands (&). Example URL:
`https://example.com?name=xavier&power=vision`.

Designing URLs is important to help convey information to developers and other users of the system. For a REST based
system, it is advisable to use nouns or noun phrases to describe resource URLs.

Routing in FastAPI is simple and very effective. A routing table (a defined list of routes) is passed to the application
when instantiating it. Each route has a corresponding endpoint. Endpoint argument can be any of the following:

+ A regular function, async function, or class method which accepts a single request argument and returns as response.
+ A class that extends the ASGI interface

Conventionally, functions are used to handle responses to a given URL path by using selected
methods (`get`, `put`, `post`, `delete`) of an instance of the FastAPI `APIRouter` class as decorators or by calling one
of the routing methods (`add_api_route`, `add_route`). The `APIRouter` class is the base class for handling HTTP
requests and this class extends the Starlette `routing.Router` class.


{{<figure src="router-structure.png" caption="The Router classes used in the mWallet application" title="Figure 1" >}}
 
The InferringSchemaRouter was designed to extend `fastapi_restful.inferring_router.InferringRouter` class. This enables the `InferringSchemaRouter` to automatically convert return types of routing functions to matching Pydantic schemas following the specified database ORM class.

{{<code lang="py3">}}


from typing import Any, List

from fastapi import Depends, APIRouter
from starlette.status import HTTP_200_OK, HTTP_202_ACCEPTED, HTTP_201_CREATED

from db.adapters import parse_to_schema
from schemas.user import UserPreference, UserDevices
from schemas.util import GenericResponse
from user import services as user_services

web_user_routes = APIRouter()


@web_user_routes.post('/user/preference', status_code=HTTP_200_OK,
                      description='List all notices sent to this user or notices which the selected user subscribes to.',
                      response_model=GenericResponse)
def create_user_preference(result_set: Any = Depends(user_services.create_user_preference)) -> UserPreference:
    return parse_to_schema(UserPreference, result_set)


@web_user_routes.delete('/user/preference/{preference}', status_code=HTTP_202_ACCEPTED,
                        description="Delete a selected user’s preference.")
def destroy_user_preference(result_set: Any = Depends(user_services.destroy_user_preference)) -> bool:
    return bool(result_set)

{{</code>}}

The `create_user_preference` function takes a result_set parameter. This parameter is injected into the function via the
FastAPI `Depends` class. The `Depends` class is the FastAPI approach in implementing Dependency Injection (a technique
in dynamically injecting a code into another code making them loosely coupled). The `result_set` can be any type of
Python data and we do not need to know what type upfront. We will simply pass the `result_set` to a `parse_to_schema`
function which has been designed to convert a range of data types into a specified Pydantic model class. In this
example, the `result_set` will be converted to the `UserPreference` data type and returned at the end of
the `create_user_preference` execution cycle. A similar approach is used with the `destroy_user_preference` which
returns a Boolean.

The server responses to HTTP requests are mapped to corresponding `Response` objects (classes that extend the
Starlette `responses.Response` class). Various response types are differentiated mainly by the specified media or mime
which they handle. For example, the `JSONResponse` class manages the media type of `application/json` while
the `PlainTextResponse` manages media type or mime of `application/text`. Other media types are mapped to their
corresponding `Response` type or subclass. See Figure 2 below for FastAPI response classes and their hierarchy. 
See Table 1 for the various response classes and corresponding mime.

{{<figure src="response-package.png" caption="Class Hierachy of FastAPI Response classes" title="Figure 2" >}}

Response classes of the Starlette responses package. FastAPI uses Starlette’s response package internally to
handle all requests.

The following response classes can be used to manage server responses.

{{<table style="table-hover" caption="FastAPI Response classes with associated MIME" title="Table 1">}}
| Class	            | Media Type                   | 	Description                                                                                             |
|-------------------|------------------------------|------------------------------------------------------------------------------------------------------------|
| Response          | `text/html`                  | Base class for managing server responses                                                                   |
| HTMLResponse      | `text/html`                  | Specialised class for managing HTML responses                                                              |
| PlainTextResponse | `text/plain`                 | Specialised class for managing text responses                                                              |
| JSONResponse      | `application/json`           | Specialised class for managing JSON responses. This is the default response used for most web applications |
| RedirectResponse  | `None`                         | Specialised class for managing redirects                                                                   |
| StreamingResponse | `typing.Optional[str]` or `None` | Specialised class for managing streaming responses example a live video stream                             |
| FileResponse      | `typing.Optional[str]` or `None` | Specialised class for managing file responses example a binary file for download                           |
{{</table>}}


## Using YAML file for defining routes

When building a web application, thoughts should be given to the design of the URL. FastAPI being built on Starlette
allows developers to use URI templating style to capture path components.

{{<code lang="py3">}}
url = '/user/{id}/payment-receipts'

{{</code>}}

Fragments of the URI template can be converted to native Python data types via convertors. There are default convertors
for `str`, `int`, `float`, `uuid.UUID` data types or a path variable.

{{<code lang="py3">}}
d1 = '/user/{id: str}/payment-receipts'  # converts id to str type
d2 = '/user/{id: float}/payment-receipts'  # converts id to float type
d3 = '/user/{id: int}/payment-receipts'  # converts id to int type
{{</code>}}

Customised convertors can be added to a web application but that is outside the focus of this book. In the mWallet
application, we will define the various URLs using a YAML file. This file will be dynamically loaded into our
application and be used to provide information on each route. Some of the data extracted from the file are shown below.

{{<code lang="yaml">}}
- route:
    name: authentication
    description:This resource will manage user activities associated with registration, login, logout, and password reset.
    base: /authentication
    endpoints:
      - access token:
          methods: [ 'POST' ]
          success: Access token generated for user with provided credentials
          error: Unable to generated access token for the user with provided credentials
          status: 201
          path:
            url: /token
            description: |
              Provide user access token to application services using credentials such as username and password
{{</code>}}

The extract from the YAML file above shows an instance of a registered route for the mWallet application. This example
is for a user authentication service. By using this approach, the development team has a working document that guides
what each API endpoint could handle. This approach also enforces the team to have a clearly defined list of URLs in line
with agreed system purpose statement. Some of the key terms used in the application `routes.yml` file are shown
below (Table 2). These entries will be used by the custom `AppRouter` and the FastAPI APIRouter class to pass routes to
the application.

{{<table style="table-hover" title="Table 2" caption="The route YML file components and description." >}}
| Route components | 	Description                                                                                                                                                                                       |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name             | The name assigned to the route                                                                                                                                                                     |
| description      | 	A short description of the route that will be displayed in the swagger documentation or provide more information about a given item.                                                              |
| success          | Message to display if the route endpoint was executed successfully                                                                                                                                 |
| errors           | Message to display if the route endpoint fails                                                                                                                                                     |
| methods          | The HTTP methods supported by the route. Typical methods are `POST`, `GET`, `PUT`, or `DELETE`                                                                                                     |
| path             | The URL for this route using URI templating                                                                                                                                                        |
| endpoints        | A collection of routes associated with a given resource. Each endpoint will have its own definition for URL path, path components, success and error messages, HTTP response status, HTTP methods. |
| status           | HTTP response status to return if the route endpoint was successful.                                                                                                                               |
| url              | A nested object that defines the url path, query and path parameters with their associated name and description. A url must be defined for each route.                                             |

{{</table>}}

We have seen earlier how to register a route in FastAPI (see Classical way of defining routes in FastAPI). We have also
defined a `routes.yml` file that is used to register a list of routes with their corresponding components. How can we
use this YML file to dynamically load routes? To answer this, we will need to discuss the `AppRouter`, and `RouteEntry`
classes and see how the mWallet uses these in the router package to register all routes of the application.

{{<code lang="py3">}}

import os
import re
from urllib.parse import urljoin

import yaml
from typing import Dict, List, Tuple, Any
from pydantic import BaseModel

import stringcase

__all__ = 'AppRouter', 'app_routes'

from util import extract_data


class AppRouter:
    # Process routes for the app by loading data from routes.yml

    __slots__ = '_data', '_named_routes', '_routes'

    def __init__(self, resources: Dict):
        if 'resources' in resources:
            data_ = extract_data('resources', resources)
            # get names of all routes. Instantiate RouteEntry for each route.
            self._data = []
            self._named_routes = []
            self._routes = []

            self._data = data_.copy()
            self._named_routes = [stringcase.spinalcase(stringcase.lowercase(v)) for y in self._data
                                  for k, v in y.items() if k == 'name']
            result = {}
            for entry in self._data:
                name = extract_data('name', entry)
                description = extract_data('description', entry)
                resource = extract_data('resource', entry)
                base_path = extract_data('base', entry)
                self._named_routes.append(name)
                endpoints = extract_data('endpoints', entry)
                for e in endpoints:
                    for k, v in e.items():
                        endpoint_name = f'{stringcase.snakecase(k)}@{name}'
                        result.update(dict(
                            endpoint=endpoint_name,
                            operation=extract_data('crud', e),
                            url_path=extract_data('path', e),
                            methods=extract_data('methods', e),
                            success=extract_data('success', e),
                            error=extract_data('error', e),
                            status=extract_data('status', e),
                            payload=extract_data('payload', e)))

                        route_entry = RouteEntry(name=name, path=base_path, resource=resource, description=description,
                                                 **result)
                        self._routes.append(route_entry)

    def find(self, name: str = None, endpoint: str = None):
        route_search = []
        endpoint_search = []
        if name and not endpoint:
            route_search = [x for x in self._routes if x.name == name]
        elif name and endpoint:
            route_search = [x for x in self._routes if x.name == name and endpoint in x.endpoint]
        if endpoint and not name:
            endpoint_search = [x for x in self._routes if x.endpoint == endpoint or endpoint in x.endpoint]

        if route_search and not endpoint_search:
            return route_search[0]
        elif endpoint_search and not route_search:
            return endpoint_search[0]
        elif route_search and endpoint_search:
            rsc_1 = endpoint_search[0]
            rsc_2 = route_search[0]
            if rsc_1.endpoint == rsc_2.endpoint:
                return rsc_1
            return False

    @property
    def routes(self):
        """
        Instance property of the list of routes registered in the application.
        :return: A list of instances of RouteEntry for each registered route
        """
        return self._routes

    @property
    def keys(self):
        """
        Instance property of the list of base routes registered in the application
        :return: list of named base routes
        """
        return self._named_routes

    @property
    def resources(self):
        """
        Provide a set of all named resources registered with the application routes
        :return: a set of registered Resource names.
        """

        rsc = (a.resource for a in self._routes if a.resource is not None)
        return set(rsc)


class RouteEntry:
    __slots__ = '_path', '_methods', '_name', '_success', '_error', '_status', '_url_path', '_payload',
        '_url_query', '_endpoint', '_resource', '_description', '_tags', '_operation', '_schema'

    def __init__(self, endpoint: str, path: str, methods: List, name: str, success: str, error: str, status: int | str,
                 url_path: str, payload: Dict = None, resource: str = None, description: str = None, tags: List = None,
                 operation: str = None, schema: Any | BaseModel = None):
        def _validate_status(s):
            if isinstance(s, int | str):
                if str(s).isnumeric() and 100 <= int(s) <= 600:
                    return True

        def _validate_payload(p):
            if isinstance(p, List | Tuple):
                return [z for z in p for x, _ in z.items() if x in ['name', 'description']]

        def _validate_methods(m):
            if isinstance(m, List | Tuple):
                return [x for x in m if x in ['GET', 'POST', 'PUT', 'DELETE', 'HEAD']]

        def _validate_url_path(base_url, up):

            url = extract_data('url', up)
            params = extract_data('params', up)
            if not base_url:
                raise ValueError('Invalid base URL provided')
            if not url:
                rsc = base_url
            else:
                rsc = base_url + url

            if params and isinstance(params, List | Tuple):
                pattern = r'{(.*)}'

                for p in params:
                    path_variable = extract_data('path', p)
                    if path_variable and isinstance(path_variable, List | Tuple):
                        for x in path_variable:
                            for k, v in x.items():
                                if k == 'name' and v:
                                    if re.search(pattern, rsc):
                                        re.sub(pattern, '{%s}' % v, rsc)
                                    else:
                                        rsc += '/{%s}' % v
            return rsc

        def _validate_tags(t: List):
            if t:
                return [x for x in t if isinstance(x, str)]

        def _validate_operation(op: str):
            if op in ['add', 'edit', 'delete', 'view', 'list']:
                return op

        self._endpoint = endpoint if isinstance(endpoint, str) else None
        self._path = path if isinstance(path, str) else None
        self._methods = _validate_methods(methods)
        self._name = name if isinstance(name, str) else None
        self._success = success if isinstance(success, str) else None
        self._error = error if isinstance(error, str) else None
        self._status = status if _validate_status(status) else None
        self._url_path = _validate_url_path(urljoin(name, self._path), url_path)
        self._payload = _validate_payload(payload)
        self._resource = resource if isinstance(resource, str) else None
        self._description = description if isinstance(description, str) else None
        self._tags = _validate_tags(tags)
        self._operation = _validate_operation(operation)
        self._schema = schema

    @property
    def path(self):
        return self._path

    @property
    def methods(self):
        return self._methods

    @property
    def name(self):
        return self._name

    @property
    def success(self):
        return self._success

    @property
    def error(self):
        return self._error

    @property
    def status(self):
        return self._status

    @property
    def url_path(self):
        return self._url_path

    @property
    def payload(self):
        return self._payload

    @property
    def endpoint(self):
        return self._endpoint

    @property
    def resource(self):
        return self._resource

    @property
    def description(self):
        return self._description

    @property
    def tags(self):
        return self._tags

    @property
    def crud_type(self):
        return self._operation

    @property
    def schema(self):
        return self._schema

    @schema.setter
    def schema(self, value):
        if isinstance(value, (Dict, BaseModel)):
            self._schema = value


base = os.getcwd()
route_path = os.path.join(base, 'routers', 'routers.yml')
app_routes = None

with open(route_path, 'r') as fh:
    try:
        data = yaml.safe_load(fh)
        app_routes = AppRouter(data)
    except yaml.YAMLError:
        raise

{{</code>}}

The `AppRouter` class serves as a registry of all API endpoints for the system. This class defines three
parameters: `routes`, `keys`, and `resources`. The routes parameter returns a list of `RouteEntry` objects that holds
information (example name, URL, description etc.) about each route. The `keys` parameter gives a list of names that are
mapped to each route. The `resource` parameter provides a set of named resources (example `Order`) which are registered
in the system. The `__init__` function of the `AppRouter` class is where data read from `routes.yml` is parsed and added
to an internal list called `_data`. Each entry from the `routes.yml` file is parsed into an instance of `RouteEntry` by
extracting attributes (`description`, `resource`, `base`, `endpoints`, `name`) from the `routes.yml` data.

The find method of the `AppRouter` class is used to verify if a given named route exists within the defined list of
routes. This function can take a name or `endpoint` parameter. If only the `name` is given, then the search is based
only on the `name` assigned to a route as defined in `routes.yml`.

{{<code lang="yaml" >}}
- route:
    name: authentication
    description:This resource will manage user activities associated with registration, login, logout, and password reset.
{{</code>}}

{{<code lang="shell" >}}
>>> app_routes.find(name='authentication')
<routers.RouteEntry object at 0x7f45e52d4860>

{{</code>}}

If only the endpoint is given then the search is based on the name of the function registered against the route.

{{<code lang="yaml">}}
- route:
    name: authentication
    description:This resource will manage user activities associated with registration, login, logout, and password reset.
    base: /authentication
    endpoints:
      - access token: 
        methods: ['POST']
{{</code>}}

{{<code lang="bash" >}}
>>> app_routes.find(endpoint='access_token')
<routers.RouteEntry object at 0x7f45e52d4860>

{{</code>}}

Notice that the endpoint is specified without spaces as this closely matches a function name in Python. Searches can
also be done by specifying the named route to which the function is assigned using the `@` symbol.

{{<code lang="bash" >}}
>>> app_routes.find(endpoint='access_token@otp')
<routers.RouteEntry object at 0x7f45e52d4860>

{{</code>}}

From the returned RouteEntry object, we can access information such as HTTP methods supported by calling property
methods; the name of the route by calling name; the type of CRUD (Create Read Update Delete) operation supported at this
API endpoint by calling `crud_type`; the description; the path and many other properties as defined in `RouteEntry`.
More importantly, we can register routes in logical groups without the need of typing all entries manually.

{{<code lang="py3">}}
from authentication.services.jwt_handler import user_login, user_logout, user_signup
from authentication.services.otp import verify_login_otp, generate_login_otp
from permissions import Permission
from routers import app_routes
from routers.api import InferringSchemaRouter

# Define the various ACL for catalogue services as scopes.

acl = Permission()

service = 'Services'
prefix = 'organization'

auth_router = InferringSchemaRouter()

r1 = app_routes.find(endpoint='user_login')
r2 = app_routes.find(endpoint='user_logout')
r3 = app_routes.find(endpoint='user_signup')
r4 = app_routes.find(endpoint='verify_login_otp')
r5 = app_routes.find(endpoint='generate_login_otp')

# Match routes to functions 
add_auth_routes = [
    (r1, user_login),
    (r2, user_logout),
    (r3, user_signup),
    (r4, verify_login_otp),
    (r5, generate_login_otp)
]

# dynamically add routes to instance of APIRouter using loaded RouteEntry objects

for entry in add_auth_routes:
    r, e = entry
    if r:
        auth_router.add_api_route(r.url_path, endpoint=e, description=r.description,
                                  status_code=r.status, methods=r.methods, response_description=r.success,
                                  name=r.endpoint)


{{</code>}}

The routes `r1` to `r5` are mapped to functions `user_login`, `user_logout`, `user_signup`, `verify_login_otp`,
and `generate_login_otp` as a list of tuples. This list is then iterated and each route and entry added to the
specified `APIRouter` instance. This way, defined routes can be managed from a centralised `routes.yml` file and changes
to the routes or associated parameter is automatically reflected in the system on reboot or start up. 
