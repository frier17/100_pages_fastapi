+++
title = 'Application services'
date = 2023-11-05T21:53:54+01:00
weight = 6
draft = true
+++

In previous Chapter, [Identifying Routes](/chapter3/), we had briefly mentioned FastAPI use of URI template to define the URL
associated with an endpoint. Path parameter can be used to pass extra information to a given API endpoint.

## FastAPI Path parameters

Path parameters or variables are declared using the left and right curly braces with the named variable within the
braces, example `{parameter}`.

Once a parameter has been defined, the value assigned to the parameter can then be passed to the associated function of
the given API endpoint. Let us revisit the previous example of [Chapter 3](/chapter3/).

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

The defined route URI is `/user/preference/{preference}` which has a path variable or parameter called `preference`.
The data type of the variable is not specified in the example above. We can specify a data type by adding a colon and
the type of data, example, `/user/preference/{preference: str}` implies the preference is expected to be a string
type. In another scenario, we could specify the data type in the associated function (in our
example, `user_services.destroy_user_preference`) and FastAPI will cast the parameter into the specified data type
before passing it to the function as shown below. In the function below, the preference parameter is expected to be of
type `str` or `UUID` and will be passed into the function by the `Path` class instance.

{{<code lang="py3" >}}
import uuid

from fastapi import Path


def destroy_user_preference(preference: str | uuid.UUID = Path(..., description="The preference identifier for uniquely filtering user’s preference.")) -> bool:
    # process preference here
    ...

{{</code>}}

The function definition shows a Path object being used to inject value into the function. This `Path` object is like
the `Depends` object introduced earlier. The `Path`, `Body`, `Query`, `Form`, `Cookies` objects are example of FastAPI
mechanism of passing request data to associated API functions. Note the ellipsis `...` that is the first argument of
the `Path` initialization. This indicates that the parameter annotated by  `Path` is mandatory. To specify an optional
parameter, the `None` keyword can be passed as the first argument. By using the `Path` class to specify and initialize
the `preference` argument, FastAPI will automatically inject value passed in the URL into the underlying function
through the `APIRouter` or specified router instance, in our example, `web_user_routes`. If a data type is specified,
then the value of the parameter is parsed to the specified data type. Where the parameter cannot be parsed into the
target data type, a HTTP parse error with the error code set to `422` will be raised.

It is important to note that `Path` parameters are usually scalar types like `str`, `int`, or a `float`. If a Pydantic
model is specified as the path parameter, then FastaAPI assumes a request body is expected and not a path variable. A
path parameter can also be given a default value. In the previous example, we specified that `preference` must be
assigned a value in the URL string by using ellipsis in our Path initialization. Alternatively, we can specify a default
value either by declaring this in the argument of the function or by specifying the default parameter of `Path`.

{{<code lang="py3">}}
import uuid
from fastapi import Path

def destroy_user_preference(preference: str | uuid.UUID = Path(default='786-rt4',
                                                               description="The preference identifier for uniquely filtering user’s preference.")) -> bool:
    # process preference here
    ...


def destroy_user_preference(preference: str | uuid.UUID = '786-rt4') ->  bool:
    # process preference here
    ...

{{</code>}}

Both cases assign a default value of '786-rt4' to the preference parameter. However, in the first case, we specify that
the preference value be retrieved from the URL path and then passed to the function as a string or UUID type. In the
later case, we specified the default value be assigned to the parameter preference. Any function
calling `destroy_user_prefence` will need to provide a value for preference else the default value will be used.

It is important to note that path parameter can only be defined once for a given route that is added to a router. Also,
associated function to an API endpoint URL should not be shared across other URLs. The example below will give a
warning.

{{<code lang="py3" >}}
# Excluded imports to simplify code

# Case A
@web_user_routes.post('/user/preference/{preference}', status_code=HTTP_200_OK, description='...',
                      response_model=GenericResponse)
def create_user_preference(result_set: Any = Depends(user_services.create_user_preference)) -> UserPreference:
    return parse_to_schema(UserPreference, result_set)


# Case B: Same endpoint function in Case A used for different URL. 
# While this may run successfully, it makes it difficult to maintain code
@web_user_routes.post('/user/modify/{preference}', status_code=HTTP_200_OK, description='...',
                      response_model=GenericResponse)
def create_user_preference(result_set: Any = Depends(user_services.create_user_preference)) -> UserPreference:
    return parse_to_schema(UserPreference, result_set)


# Case C: Matches URL of Case A though with different endpoint function
# FastAPI will give warning here. Both A and C will resolve as equivalent or identical endpoint.
@web_user_routes.post('/user/preference/{preference}', status_code=HTTP_200_OK, description='...',
                      response_model=GenericResponse)
def create_user_preference(result_set: Any = Depends(user_services.change_user_preference)) -> UserPreference:
    return parse_to_schema(UserPreference, result_set)

{{</code>}}

Both `Case B` and `Case C` are not valid and should not be used.

## FastAPI Query parameters

API endpoint functions with parameters that are not defined in the path are automatically converted to `Query`
parameters. In the URL or URI template, the query is a key-value pair following the question mark. Example
URL: `https://example.com?name=xavier&power=vision`. The query parameters are `name` with the string value `xavier`
and `power` with the string value `vision`. In FastAPI, `Query` parameters by default are strings but can be parsed to other
Python types using type hint.

{{<code lang="py3">}}
result_set = [{'color': 'yellow', 'font': 'Arial'}, {'color': 'black', 'font': 'Courier'}]

# Case A
@web_user_routes.post('/user/preferences', status_code=HTTP_200_OK, description='...', response_model=GenericResponse)
def list_preferences(skip: int = 0, limit: int = 0) -> List[UserPreference]:
    return parse_to_schema(UserPreference, result_set[skip:limit])

{{</code>}}

In the example, above (`Case A`), the `result_set` variable is a hypothetical list of mappings having the keys color and font.
Let's assume each mapping in the list can be parsed to the `UserPreference` by the `parse_to_schema` function. Our query
parameter, `skip` and `limit` will automatically be injected by FastAPI into the function, `list_preferences` when a
user visits the URL `http://domain.com/user/preferences?skip=5&limit=10`. Note that the value in the URL is a string "5"
and not integer 5. However, since a type is specified in the function definition, the string is parsed, and an integer
value is passed into the function.

The parameter value can be set in the URL or in the function definition. If set in the URL, then the default value in
the function will be disregarded and the URL value will be parsed and used. If no value was set in the URL, then the
function default value will be used. To specify a query parameter as optional, set the default value to `None`.
Alternative approach is to specify a default by using the `Query` class default keyword.

{{<code lang="py3">}}

# Excluded imports to simplify code

result_set_a = [{'color': 'yellow', 'font': 'Arial'}, {'color': 'black', 'font': 'Courier'}]
result_set_b = [{'color': 'yellow', 'font': 'Arial'}, {'color': 'black', 'font': 'Courier'}]
result_set_c = [{'color': 'yellow', 'font': 'Arial'}, {'color': 'black', 'font': 'Courier'}]


# Case A
@web_user_routes.post('/user/preferences', status_code=HTTP_200_OK, description='...', response_model=GenericResponse)
def list_preferences(skip: int = 0, limit: int = None) -> List[UserPreference]:
    return parse_to_schema(UserPreference, result_set_a[skip:limit])


# Case B
@web_user_routes.post('/user/preferences', status_code=HTTP_200_OK, description='...', response_model=GenericResponse)
def list_preferences(skip: int = Query(...), limit: int = Query(default=None) -> List[UserPreference]:
    return parse_to_schema(UserPreference, result_set_b[skip:limit])


# Case C
@web_user_routes.post('/user/preferences', status_code=HTTP_200_OK, description='...', response_model=GenericResponse)
def list_preferences(skip: int = Query(...), limit: int = Query(None) -> List[UserPreference]:
    return parse_to_schema(UserPreference, result_set_c[skip:limit])


{{</code>}}

The value of limit in Case B, and C are equally set to None as both expressions are equivalent.

#### ℹ️ Tips and Tricks - slicing result

> The slice operation in Python is used to retrieved values from a list or other sequences that supports slicing. This
> technique allows for selecting from index skip to index limit (values inclusive). A similar technique is used for actual
> database results when working with ORMs like `SQLAlchemy` and Django `Queryset`. When using slicing, ensure the skip and
> limit values are within range of the sequence. Using values out of range will raise an `IndexError`.

`Query` parameters can also be defined as mandatory fields aside being default or optional values. To define a parameter
as mandatory, use the ellipsis (`...`) as the first argument to the `Query` class initialization call. Let’s update our previous
example to show this.

{{<code lang="py3">}}
# Excluded imports to simplify code

result_set = [{'color': 'yellow', 'font': 'Arial'}, {'color': 'black', 'font': 'Courier'}]


@web_user_routes.post('/user/preferences', status_code=HTTP_200_OK, description='...', response_model=GenericResponse)
def list_preferences(skip: int = Query(...), limit: int = None) -> List[UserPreference]:
    return parse_to_schema(UserPreference, result_set[skip:limit])

{{</code>}}

In the updated example, the `skip` parameter is now mandatory. Every call to the URL must provide a `skip` parameter.
Calling the `list_preference` function without the skip parameter will raise an error. Another way of making a parameter
mandatory or required, is to remove any default value from the function declaration as shown below.

{{<code lang="py3">}}
# Excluded imports to simplify code

result_set = [{'color': 'yellow', 'font': 'Arial'}, {'color': 'black', 'font': 'Courier'}]

# Case B
@web_user_routes.post('/user/preferences', status_code=HTTP_200_OK, description='...', response_model=GenericResponse)
def list_preferences(skip: int, limit: int) -> List[UserPreference]:
    return parse_to_schema(UserPreference, result_set[skip:limit])


# Case B
@web_user_routes.post('/user/preferences', status_code=HTTP_200_OK, description='...', response_model=GenericResponse)
def list_preferences(skip: int = Query(...), limit: int = Query(...)) -> List[UserPreference]:
    return parse_to_schema(UserPreference, result_set[skip:limit])
{{</code>}}

In the above example, the `skip` and `limit` parameters are required. Notice the default values for both parameters have
been dropped. Both cases shown above will set the `skip` and `limit` parameters as required parameters of
the `list_preference` function.

A URL may define both path and query parameters. In fact, multiple parameters may be defined.
Example, `http://localhost/user/{user}/type/{order}?skip=5&limit=10` can be interpreted as a link for requesting
for a given user’s order of a specified type with value defined by `{order}`. The Path parameters `{user}` and `{order}`
can be used to identify a given user and the type of order. The Query parameters `skip`, and `limit` can be used as
indices for selecting a range of results.

## FastAPI Body parameters

Thus far we have been sending data as scalar values to our server or localhost. What if we needed to send a more complex
data structure? FastAPI provides the `Body` parameter for sending data via HTTP requests. The `Body` class like
the `Path` and `Query` can be used in function definitions to pass data as Python dictionary by default (where a schema
is not specified) or parsed into a specified Pydantic model or schema. This class takes similar keyword arguments like
the `Query` and `Path` parameter. Also, `Body` parameters can also be defined as optional or required just like we did
with `Query` parameters.

{{<code lang="py3">}}
from fastapi import Body


def user_signup(request: Request, data: dict = Body(..., description="")) -> Any:
    # create a new user
    s = get_session()
    with s.begin():
        if not ('password' in data and ('email' in data or 'username' in data)):
            raise HTTPException(status_code=500, detail='Invalid parameters')

        if token := create_user(data, session=s):
            timestamp = current_timestamp()
            ip = request.client.host
            system = request.headers.get('User-Agent')

            EventBus.broadcast(key=sign_up_signal.identity,
                               data={'ip': ip, 'timestamp': timestamp, 'system': system, 'email': data.get('email')})
            return token
    return False


{{</code>}}

In the example, above, the first parameter to the user_signup function is a request variable of type `Request`.
The `Request` class can be imported from the FastAPI package or `starlette.requests` package. Both `Request` are
equivalent as the `Request` class from FastAPI is of type `starlette.requests.Request` class. In our example, the
request object is used to access data of the user agent (example, the operating system used, the browser agent used and
the IP address of the client). When a request object is passed to a function, all request components such
as `URL`, `Header`, `Cookies` etc. can be accessed via the object. In the mWallet application, a broadcast mechanism was
designed to enable application send events to various functions or classes that execute some code with the parameters
passed to the `EventBus.broadcast` method. This approach was adopted to decouple router functions from other application
level functions that may run a background task or other associated operations. In the code snippet above, the `EventBus` will
trigger the `sign_up_signal` function handler that will send a message to the specified recipient using the email address as provided
in `data`. the `sign_up_signal` handler will also log any error in sending an email alongside with the IP, timestamp, 
and system information at the time of registration.

{{<code lang="py3" >}}
from fastapi.requests import Request
from fastapi.responses import Response


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    request = Request(scope, receive)
    body = 'Some random text to display!'

    # To view URL 
    url = request.url

    # To view base url
    base_url = request.base_url

    # To view headers
    headers = request.headers

    # To view queries sent in the request
    queries = request.query_params

    # To view path parameters in the request
    path_parameters = request.path_params

    # To view cookies
    cookies = request.cookies

    response = Response(body, media_type='text/plain')
    await response(scope, receive, send)

{{</code>}}

Note that the FastAPI `Request` class provides a `body` property which returns a mapping or dictionary. Using the `Body`
parameter and type hinting, the value returned from `Request.body` parameter can be set to an instance of a specified
pydantic model or data type.

{{<code lang="py3" >}}
# Excluded some imports to simplify code

from pydantic import BaseModel


class UserLoginData(BaseModel):
    password: str
    username: str
    nonce: int | str = None


def user_signup(request: Request, data: UserLoginData = Body(..., description="")) -> Any:
    # create a new user
    s = get_session()
    with s.begin():
        if not ('password' in data and ('email' in data or 'username' in data)):
            raise HTTPException(status_code=500, detail='Invalid parameters')

        if token := create_user(data, session=s):
            timestamp = current_timestamp()
            ip = request.client.host
            system = request.headers.get('User-Agent')

            EventBus.broadcast(key=sign_up_signal.identity,
                               data={'ip': ip, 'timestamp': timestamp, 'system': system, 'email': data.get('email')})
            return token
    return False

{{</code>}}

When a `Body` parameter type is specified in function declaration, FastAPI automatically adds this to the web API
documentation. If a schema is defined, then the documentation will provide information on data structure and any other
information added in the schema by the developer.

{{<figure src="fastapi-schema.png" caption="A sample FastAPI swagger documentation for a defined model" title="Figure 1" >}}

Just as we can have Path and Query parameters, we can also have `Path`, `Query`, and `Body` parameters. We can equally
have multiple `Body` parameters in a function declaration. Note that where multiple `Body` parameters are used, a
dictionary is passed to the function with the keys as name of the `Body` parameters.

{{<code lang="py3">}}
def approve_payment(token: str = Security(get_user_access), schedule: str = Path(...), approval: Dict = Body(...),
                    receipt: Dict = Body(...), confirmation: Dict = Body(...)) -> Any:
    return approval_services.approve_payment(usid=token, schedule=schedule, receipt=receipt, approval=approval,
                                             confirmation=confirmation)


{{</code>}}

The `approve_payment` function takes multiple parameters using the `Path` and `Body` classes. The schedule parameter
will be a string passed from the URL path to the function. The `approval`, `receipt`, and `confirmation` variables will
be parsed into a dictionary with matching keys and values as dictionaries from the request body as shown below.

{{<code lang="py3">}}
# Result of combined parameter when using multiple Body parameters.
{
    'approval': {},
    'receipt': {},
    'confirmation': {}
}
{{</code>}}

#### 📚 Further Read – Path, Query, Body parameters

> In the examples of this Chapter, the `typing.Annotated` class was not used with `Query`, `Path`, or `Body` parameters.
> The `Annotated` class was introduced after Python 3.6 and some FastAPI code in the public may still use earlier format
> without `Annotated`. These are still equivalent to the functions using `Annotated` class.
> __FastAPI tutorials__
> + 📖 [FastAPI Path parameter tutorial](https://fastapi.tiangolo.com/lo/tutorial/path-params/)
> + 📖 [FastAPI Query parameter tutorial](https://fastapi.tiangolo.com/lo/tutorial/query-params/)
> + 📖 [FastAPI Body parameter tutorial](https://fastapi.tiangolo.com/lo/tutorial/body/)
