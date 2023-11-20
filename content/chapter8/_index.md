+++
title = 'Testing'
date = 2023-11-05T22:08:25+01:00
weight = 10
draft = true
+++


In my opinion, the act of testing is one of those good things which a lot of people agree should be done yet it is often
not done. I believe this is especially true amongst rookie developers. Testing should not only be encouraged but also
needs to be well taught. Too often, the wrong parameters are used for testing and tests may be focused on the wrong
things. As an example, certain aspects of FastAPI are thoroughly tested and doesn’t need to be tested again in a web
application built on FastAPI. Knowing what area of an application that should be tested is a skill that should be sought
after. Also, knowing what tests to write is equally important. The idea of testing is not to have a successful result or
passing test. The idea of testing is to have a declaration of what the system does. Tests in this sense should be seen
as markers or targets which the developer is working towards. The target should be clearly defined and reasonably static
as tests are reference points. As the application requirement changes within reasonable limits, the tests can be
modified where applicable. Again, the aim is not to get a passing test but to ensure the right system is being tested
and built.

Let’s consider testing the mWallet application. To demonstrate various test approaches, we will use unit test approach
for some aspect of our system. We will adopt the end-to-end approach for testing for selected section of the system that
deals with CRUD operations. One of the good testing frameworks to use is the pytest. The pytest package is supported by
FastAPI and was used in the examples for this project. Python doctests were also written for functions that provide
specific services for our path operations. Using doctests for our functions is a good way of documenting and testing our
code. Earlier code samples have minimised or excluded comments and doctest for simplicity before introducing testing.
Readers are encouraged to document code and adopt doctest for showing samples of how to use their code. This is a good
practise and should be done as much as possible across projects. Lastly, we employed the hypothesis and faker package in
generating random values for testing our application and seeding our database. The hypothesis package will be used to
see possible values that can cause system failures, while the faker package will be used to generate sample data by
which the database records will be dynamically generated. We showed a simple use case of the faker and hypothesis
packages for testing later in this Chapter.

## Unit testing using pytest

We had earlier defined the application purpose statement and from this we defined a skeletal test suite. Let us revisit
our previous attempt in defining our tests.

{{<code lang="py3" >}}

from unittest import TestCase


class TestTransaction(TestCase):

    def setUp(self) -> None:
        ...

    def tearDown(self)- > None:

    ...


def test_place_order(self) -> None:
    ...


def test_satisfy_order(self):
    ...


def test_transaction_history(self):
    ...


def test_current_orders(self):
    ...


{{</code>}}

From the above we define our system limits or targets:

+ Users can place order
+ Users can have other users or agents satisfy an order
+ User can view transaction history
+ User can view open or current orders

Each of these features may comprise one or more sub processes which in turn may need to be tested. Theoretically,
testing each of these features would be testing our system from end to end. We would, however, adopt an approach in
which selected aspect of our system will be tested as a working unit to ascertain if these units are fit to use while
other aspect of the system will be tested from end-to-end to see overall system performance.

For example, the `test_place_order` will require a valid user instance, data to initialize an order, and valid agents (
registered users) to be available before a successful pass result can be obtained. We can mock these objects allowing 
us to focus on testing our core processes. Alternatively, we can test selected core services, that if they fail, we are
certain our place order feature will be non-operational. Using this approach, we test the critical services of our
system and not spend limited resources and time on every component, especially repeated components across various
features. With this in view, let us expand our tests to cover key processes used in our listed features.

To manage our different tests, we define a tests package and test files in this package. Each file is prefixed with the
`test_` word to enable pytest automatically pick these files and run the tests defined in them. This autorun feature can
simplify managing and running tests for the developer. To use the pytest for testing, we must first have it installed on
our working environment. The package can be installed using the pip command.

{{<code lang="bash">}}
$ pip install -U pytest

{{</code>}}

Note that although we use the unittest.TestCase class, the test can still be executed using pytest. We can also run our
tests using the Python unittest package. With our tests package define, we define a `test_task_runner.py` script for
testing running queries and other operations on separate threads. We can also define a `test_crud.py` script for testing
our CRUD operation functions like `model_create`, `model_retrieve`, `model_destroy`, `model_update`, and `model_list`.
Having the CRUD operation tests reduces testing of classes such as `ManageResource` which relies on these functions. We
can also expand on our `place_order` test in our `test_order.py` script. However, for demonstration purposes, we will
show tests for our threaded operations and a sample call to a catalogue endpoint.

{{<code lang="py3" >}}
import random
import timeit
from string import Template

from faker import Faker

from bhis_claims import settings
from db import get_session
from messages import messages
from models.administration import Group, UserType
from util import thread_runner, delegate_task, generate_rsid, current_timestamp
from util.email import SesMailSender, SesDestination

fake = Faker()


@thread_runner(workers=settings.DEFAULT_THREAD_WORKERS)
def add_group():
    data = [dict(
        scope=random.choice([x for x in UserType.get_scopes()]),
        description=fake.word(),
    ) for _ in range(5000)]
    s = get_session()
    with s.begin():
        result = s.bulk_insert_mappings(Group, data)
        s.commit()
        yield result


def test_thread_fetch():
    start = timeit.default_timer()
    a = add_group()

    end = timeit.default_timer()
    assert a

    print(f'{end - start}')


def test_delegate_task():
    email = settings.TEST_EMAIL
    timestamp = current_timestamp()
    usid = generate_rsid()
    role = UserType.BASE_USER.as_role()

    destination = SesDestination(tos=[email])

    start = timeit.default_timer()
    a = delegate_task(SesMailSender().send_email, use_max=True, subject=messages.otp_generated_subject,
                      arn=settings.AWS_SES_MAILER_ARN,
                      source=settings.APP_FROM_EMAIL,
                      destination=destination,
                      text=Template(messages.register_officer_text).safe_substitute(rsid=usid, email=email,
                                                                                    timestamp=timestamp, role=role),
                      html=Template(messages.register_officer_html).safe_substitute(rsid=usid, email=email,
                                                                                    timestamp=timestamp, role=role))

    end = timeit.default_timer()
    assert a

    print(f'{end - start}')

{{</code>}}

The `test_task_runner.py` script has shown above has a `test_thread_fetch`, `test_delegate_task`, and a `add_group`
function which is decorated by the `thread_runner` decorator. The `add_group` function simply generate data for
initializing `Group` model. The function returns a generator object which is further process by the `thread_runner`
decorator before returning a result to a caller. The `test_thread_fetch` measures the time for executing a call
to `add_group` function. The aim of the test is not to compare the time of execution but to validate that
the `thread_runner` decorator operates as designed and measure the duration taken to execute a decorated function. The
time measured for a non-decorated function can be compared for analysis, but this is outside the scope of this Chapter,
hence is not shown in the example. The reader can explore speed comparison. Time differences above ten percent (10%) can
be taken as positive improvement to the function running on multiple threads against function not running on multiple
threads. It is important to note the `thread_runner` decorator is designed for IO bound processes example, processes
that read a file or from a network. In our test example, we used the `thread_runner` for a function that writes to a
local database. A similar result should be obtained for a remote database with slight variations obtained from network
latency. The `test_delegate_task` is like the `test_thread_fetch` in that multiple threads are used to execute a
function. In this test scenario, the AWS email service (SES) is initialized and used to send email to a
predefined `TEST_EMAIL` address. This process is also an IO bound process and the duration for executing the code is
measured and printed on screen. Test results checked to positive outcome as the `SesMailSender.send_email` returns a
message ID on success or raises an error on failure.

To run the test suites, we can simply call `pytest` from a terminal. Ensure the current working directory is set to the
project root folder.

{{<code lang="bash" >}}

$ pytest
=========================== test session starts ============================
..

{{</code>}}

Successful tests are represented by a dot (`.`) while failed test are represented by the `F` alphabet. Again, the
importance of the test is not to have all successfully results. The reader is encouraged to plan their tests and ensure
their tests covers important behaviours of their system.

Our test example focuses on testing aspects of our system and not testing the data flow from the user end. Let us define
simple test for showing system behaviour from the user’s perspective. We will perform these tests using
the `FastAPI.testclient` package. The `TestClient` class is used to initialize a client that is used in making HTTP
requests to our endpoints. We will use this client to post or get data for our chosen endpoint in the respective tests
covered. From the client calls, we can obtain a corresponding HTTP response. This response will have attributes such
as `status_code`, `content`, and methods defined in the FastAPI `Response` class. The `TestClient` depends on
the `httpx` package. To use the `TestClient` in our project, we will first install the `httpx` package.

{{<code lang="bash" >}}

$ pip install httpx

{{</code >}}

We install the httpx package using the pip install command. We can then define test functions that uses the TestClient
class to evaluate our system.

{{<code lang="py3" >}}

from fastapi.testclient import TestClient
from faker import Faker
from hypothesis import given, strategies as st

from catalogue.services.catalogue import basic_search

from .main import app

fake = Faker()

client = TestClient(app)


def test_search():
    name = fake.name()
    start = fake.date_time()
    result = client.post('/basic_search', data={'name': name, 'period_start': start})

    assert result.status_code == 404


@given(name=st.text(), start=st.datetime())
def test_add_catalogue(name, start):
    result = client.post('/catalogues', data={'name': name, 'period_start': start})

    assert result.status_code == 201

{{</code>}}

We can also run the tests using `TestClient` by calling pytest to discover our defined tests and run them like we did
for previously. In our `test_add_catalogue`, we introduced a simple use case for Python `hypothesis` package. The name
and start parameter are initialized from random data generated from the `hypothesis` `strategies.text()`
and `strategies.datetime()`. These method call will generate Python instance of the `str` and `datetime` classes.
Several `name` and `start` values will be generated until an exception is raised due to mismatch parameters for our
method call. If no failed value is gotten, then our test will simply run and assets the returned `status_code`. Failing
values generated from the strategies can be used to improve error handling, data validation and similar techniques that
improves the robustness of our code.

🔍

> Several methods exist for testing an application. Irrespective of the approach use, tests should be well planned and
> treated as part of the application itself and not a secondary activity. Sample tests given in this book is only a
> premier to testing and should not be taken as an exhaustive guide on testing. The reader is encouraged to explore more
> materials or tutorial on testing.
> + 📖 [Pytest tutorial](https://docs.pytest.org/en/7.3.x/getting-started.html)
> + 📖 [Doctest – Test interactive Python Example](https://docs.python.org/3/library/doctest.html)
> + 📖 [Hypothesis – Quick start guide](https://hypothesis.readthedocs.io/en/latest/quickstart.html)
