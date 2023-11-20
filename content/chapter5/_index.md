+++
title = 'The business domain entities and database layer'
date = 2023-11-05T21:57:05+01:00
weight = 7
draft = true
+++


We had earlier identified some key resources for the application. In this Chapter, we will explore the use of pydantic
models (generally called schemas), the SQLAlchemy Object Relational Mapper (ORM) and Core to manage database
connections, queries, and storage. To build the right system, we need to ensure representations of users, systems, ideas
or other concepts are well captured in our design. In some project teams, Entity Relational Diagrams (ERD) are used to
capture and debug database model of the system in view. An ERD is a type of flow chart that shows how entities relate
with one another in a system. Going back to our example in Chapter 2, we have listed certain resources and their
corresponding operations. These resources are entities, such as Order, User, Asset, and Wallet.

🔍
> The term Resource is often used in describing objects, data or other information that is accessible via HTTP methods
> when using a REST API. Resource may be mapped to an actual database table through an Object Relational Mapper (ORM) such
> as SQLAlchemy.

We may add other entities as is obtainable with a real-world cryptocurrency exchange. However, for simplicity, we will
use our earlier named entities. We will not provide a detail ERD as that is beyond the scope of this book. We will,
however, group these entities into logical categories mapped to modules in our database package named db. The db package
will also provide utility functions to set up connection to our underlying database, define queries for performing CRUD
operations, and manage sessions to databases using separate threads to speed database operations.

🔍
> A common pattern in REST application is to manage resource through defined operations that Create, Read, Update, or
> Delete a resource. Each of these operations are associated with an HTTP method.
> + Create is achieved using HTTP `POST` method with a successful response of `201`.
> + Update is achieved using HTTP `PUT` or `PATCH` method with a successful response of `200`, `201` or `204`
> + Read is achieved using HTTP `GET` method with a successful response of `200`.
> + Delete is achieved using HTTP `DELETE` method with a successful response of `200`, `202`, `204`.

The FastAPI framework does not come with an inbuilt database ORM package like the Django framework. Instead, developers
can use any database package of their choice. This flexibility allows developers to integrate and use packages they are
already familiar with instead of learning a new ORM syntax. Before we dive deep into the database systems, let us
further explore the pydantic model and how FastAPI validates data from users or send response to users via the pydantic
model.

## Defining and Generating schemas

Pydantic package provides data validation and management using Python type hint. The `BaseModel` class is the primary
means by which other models can be created and then validated by pydantic at runtime. Arbitrary data can be used to
initialize defined classes that extends the `BaseModel`. Pydantic will validate that the data received matches the
defined data types of the selected model class. If the validation process fails, a validation error with feedback is
raised. Data received from users via HTTP requests can be passed to a pydantic model and then validated as described.
This simplifies the validation process and improves the overall performance of the web application. In the mWallet
application, a collection of pydantic models were defined in the `schemas` package. For simplicity and ease of
maintenance, schemas were designed to closely follow the database ORM classes for which they validate data.

In this book, we will call models that extends the pydantic BaseModel classes as schemas while models that extends the
SQLAlchemy’s database ORM model simply as model. Where the word schema is not used, or its usage is considered
ambiguous, or may imply something else, then the phrase pydantic model will be used. In essence, `ModelA` represents a
database ORM class that is used to save data into `table_a`. `SchemaA` represents a schema that is used to validate data
to ensure it matches definition that can be saved into database via `ModelA`. With this relationship between schema and
model, we can explore the various schemas and later the models and associated operations of the mWallet application.

{{<figure src="model-schema-setup.png" title="Figure 1" caption="Vatlidation and database data flow in a sample FastAPI application"  >}}


## Defining Schemas

We had defined our schemas to be types derived from pydantic `BaseModel` class. The class definition of this type can
declare any arbitrary field type and will automatically inherit properties of the `BaseModel`. Some base properties
inherited includes but not limited to the attributes shown below (Table 1).

{{<table style="table-hover" title="Table 1" caption="Some selected pydantic BaseModel attributes">}}
| Schema Attribute	                                | Description                                                                                                                                                                                                                                                                                                                                                                                 | 
|--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dict()`                                         | returns a dictionary of the fields defined in the schema with their corresponding values                                                                                                                                                                                                                                                                                                    |
| `json()`                                         | returns a JSON string representation of the schema `dict()` result                                                                                                                                                                                                                                                                                                                          |
| `copy()`                                         | returns a shallow copy of the schema by default. If a deep copy is required, then set the keyword `deep=True`. Keywords `include`, `update`, and `exclude` can also be specified. The include parameter is a set of fields to keep in the returned result; the `update` is a dictionary of fields to change upon `copy`; `exclude` is a set of fields to remove from the result upon `copy` |
| **Methods to parse data**                        |
| `parse_obj()`                                    | a utility for loading any object into a model with error handling if the object is not a dictionary                                                                                                                                                                                                                                                                                         |
| `parse_raw()`                                    | a utility for loading strings based on formats or patterns                                                                                                                                                                                                                                                                                                                                  |
| `parse_file()`                                   | Similar to `parse_raw()` but for file identified by path parameter. If the content_type of the file is not specified, then it is inferred from the file extension. This function passes content read from the file to the `parse_raw()` and then returns a schema if the operation is successful.                                                                                           |
| **Methods to export schema or create instances** |
| `from_orm()`                                     | loads data into a model from an arbitrary class example a SQLAlchemy result                                                                                                                                                                                                                                                                                                                 |
| `schema()`                                       | returns a dictionary representing the model as JSON Schema                                                                                                                                                                                                                                                                                                                                  |
| `schema_json()`                                  | returns a JSON string representation of schema() result                                                                                                                                                                                                                                                                                                                                     |
| `construct()`                                    | a class method for creating models without validating the data used                                                                                                                                                                                                                                                                                                                         |
| **Special pydantic model fields**                |
| `__fields_set__`                                 | Set of names of fields of the pydantic model set during class initialisation                                                                                                                                                                                                                                                                                                                |
| `__fields__`                                     | A dictionary of the pydantic model's fields                                                                                                                                                                                                                                                                                                                                                 |
| `__config__`                                     | This attribute references the configuration class for the pydantic model. It is a general practise to use the class Config to define settings instead of directly modifying `__config__`.                                                                                                                                                                                                   |
{{</table>}}

Now, lets start exploring some examples adapted from the mWallet project.

{{<code lang="py3">}} 
from enum import Enum
from typing import Optional

from pydantic import BaseModel, Field


class OrderType(Enum):
    OPEN = 0
    CLOSED = -1
    SATISFIED = 1
    PARTIAL = 2


class AssetType(Enum):
    BITCOIN = 2
    DASH = 3
    DOGECOIN = 5
    LITECOIN = 7
    NAIRA = -1


class Order(BaseModel):
    order_type: OrderType = None
    asset: AssetType = Field(default=AssetType.BITCOIN)
    start: float = Field(..., description="The order start date and time represented as epoch time")
    expected_end: Optional[float] = Field(None, description="The order start date and time represented as epoch time")
    memo: Optional[str] = Field(None, max_length=128)

{{</code>}}

We defined two enumerator types `AssetType` and `OrderType`. These enumerators are used to define the data types of
asset and `order_type` in our `Order` schema. When initializing an instance of this schema, pydantic will expect a value
of type `AssetType` to assign to the asset attribute and a value of type `OrderType` to assign to the `order_type`
attribute. Also notice how the `Field` class is used to define attributes on the schema. A default value for a schema
attribute can be set by assigning the keyword default to the value desired. Where `None` is used, the attribute is set
as optional. Alternatively, the `Optional` type from Python typing package can be used. All attributes set to None are
by default optional. When the first attribute of the Field class is an ellipsis (`...`) the attribute is marked as
required. If the `Order` class is being initialized without a corresponding start data, then a `ValidationError` will be
raised.
Our `Order` schema takes scalar types and nested `Enum` types. In some cases, we may have a more complex schema in which
other schemas are used to define an attribute.

{{<code lang="py3">}}
# Excluded some imports to simplify code

from decimal import Decimal

from util import generate_transaction_ref

from typing import Optional, Dict, List


class Transaction(BaseModel):
    order: Order
    tranx_ref: str = Field(default=generate_transaction_ref, max_length(256))
    tranx_id: str
    inputs: str
    outputs: str
    confirmations: int
    meta: Optional[Dict]

    class Config:
        arbitrary_types_allowed = True
        orm_mode = True


class CryptoAddress(BaseModel):
    address: str = Field(..., max_length(64))
    network: str = Field(max_length(32))
    is_default: bool = Field(default=False)
    address_balance: Decimal
    label: Optional[str] = Field(None, max_length(50))
    description: Optional[str] = None


class UserAddress(CryptoAddress):
    account_number: str
    account_key: bytes

    class Config:
        orm_mode = True

{{</code>}}

We now introduce a `Transaction`, `CryptoAddress`, and `UserAddress` schemas. The `Transaction` schema models a
blockchain transaction with attributes to capture the blockchain transaction hash as `tranx_id`, the input address (
sender) involved in the blockchain transaction, the output address (receiver) in the blockchain transaction, and number
of confirmations in the blockchain. Any other transaction information may be captured as a mapping of key-value pair
using the meta attribute. We equally define a `CryptoAddress` schema to capture a given cryptocurrency address. We will
not model an HD wallet or multikey wallet as an address will serve the purpose of our exchange. Notice our UserAddress
extends the CryptoAddress schema. Pydantic models like other Python classes can be extended or composed in other
classes. In the `Transaction` schema we have an attribute order of type `Order` schema. If arbitrary types are used in
our schema, this information needs to be passed to the underlying validator to enable proper parsing of data. We can
configure the needed attribute or flag for this purpose using the class `Config`
attribute `arbitrary_type_allowed = True`. We can also define an orm_mode flag using the Config class as shown in
the `UserAddress` schema. This will notify pydantic that our schema can receive data from a Python object that is
possibly an instance of a database ORM model. When the `orm_mode` is enabled, the `from_orm` method can be used to parse
an instance of our ORM into an instance of our schema.

### Generating schemas

In some scenarios, it may be desirable to dynamically generate a pydantic model that closely matches our defined
database. As an example, supposing we have an existing database that interfaces with some legacy code. We are required
to build a modern web API using FastAPI and not required to modify the underlying database. For our scenario, we are
also required to use SQLAlchemy as this already works well for the existing code base. With this in view, we could spend
hours writing codes that define our schemas, or we can rely on the power of SQLAlchemy and the Python type class to
generate the schemas we need. Let us consider the latter approach as this will enable us quickly to deliver on our
project.

The SQLAlchemy has feature that supports connecting to a database and loading ORM classes from the defined database
connection. From this feature we can generate and save to file a set of model classes that closely map to our underlying
database tables. With our models now in place, let us generate schemas. We will rely on the `type` metaclass to generate
abstract classes that extends pydantic `BaseModel`. We defined this process in the function `make_schemas`.

{{<code lang="py3">}}

import stringcase

from typing import TypeVar, Dict, Optional, List, Tuple

from pydantic import BaseModel, BaseConfig

from db import Base

ModelT = TypeVar('ModelT', bound=Base)


def make_schema(model: ModelT, parents: List | Tuple = None, defaults: Dict = None, name: str = None):
    table_columns = model.__table__.columns
    table_fields = table_columns.keys()
    table_schema = table_columns.items()
    annotations = {x: Optional[y.type.python_type] for x, y in table_schema}
    if defaults:
        defaults = {x: y for x, y in defaults.items() if x in table_fields}
    else:
        defaults = {}
    if name:
        name = stringcase.pascalcase(name)
    else:
        name = stringcase.pascalcase(model.__name__ + 'Schema')
    config = BaseConfig
    config.arbitrary_types_allowed = True
    config.orm_mode = True
    values = {'__annotations__': annotations, 'Config': config}

    values.update(defaults)

    if parents:
        bases = (BaseModel,)
        allowed = (x for x in parents if issubclass(x, BaseModel))
        bases += tuple(allowed)
    else:
        bases = (BaseModel,)
    dyn_schema = type(name, bases, values)
    return dyn_schema

{{</code>}}

The `make_schema` function takes a SQLAlchemy ORM mapper class, `ModelT`, a list of base classes which the final schema
should extend, a dictionary of default values the schema should have, and the name of the final schema being generated.
Using the model class, we retrieve a set of table columns which are defined as attributes on the model class. This is
done by calling the mapper property `__table__` and then calling the columns property of the table object returned
by `__table__`. From the returned list of columns, we can retrieve only the column names by calling the `keys()` and
the column definitions by calling the `items()` method. Now we have both names for the underlying table columns and the
type of data the column should store. We can use this information to define the fields of our schema and the datatype
which should be assigned to a given field. We define an `annotations` variable by using Python dictionary comprehension.
We can further process the default values for the schema passed to the `make_schema` function. If the defaults
dictionary is not empty, we update it by taking only keys that have same names as our table columns - as we would want
our final schema to be identical to our model for simplicity. This `defaults` dictionary is then combined with
the `annotations` dictionary to generate a dictionary of all fields and configuration with their corresponding values.
Notice we programmatically set the `orm_mode` flag in our configuration dictionary. By using the Python `type`
metaclass, we can generate a new class that extends pydantic `BaseModel`. The `type` class can take three parameters
namely the name of the new class being generated, the bases which is a list of parents to inherit from, the default
values of member attributes or fields of the newly generated class. Once these parameters are defined, we can return a
schema class that extends `BaseModel` and any other schema specified via parents parameter. Our schema class will also
have attributes with values as specified by `defaults`. The datatype for each field will be set by the `annotations`
dictionary and each datatype will match the specified Python datatype used in defining our SQLAlchemy models.

#### 📚 Further Read – Python type and metaclass

> Mataclass is a template by which other classes can be defined. It is a blueprint that describes how other classes can
> be constructed. A class definition will create a class name, a dictionary of class attributes, and list of parent
> classes. The metaclass is used to manage the creation of these components of a class and manage the creation of the
> class through the `__new__` method. The class `__init__` method can be used to initialize instances of a class.
> The `__init__` is not a constructor as obtainable in languages like Java. Rather, it is an instance method used to set
> instance attribute after being created by the metaclass `__new__` method.
> Metaclasses are to classes as classes are to objects. Metaclasses in Python are used to achieve a design goal in which
> base classes can constrain the behaviour of their children’s classes.
> + 📖 [Real Python Tutorial on metaclasses](https://realpython.com/python-metaclasses/)
> + 📺 [Example video on mataclasses from YouTube](https://www.youtube.com/watch?v=cKPlPJyQrt4&pp=ygUScHl0aG9uIG1ldGFjbGFzc2Vz)


In the mWallet application, the `make_schema` class is used to specify the `response_model` variable dynamically if we have
the name of the target ORM model.

## SQLAlchemy and the Object Relational Mapper (ORM)

SQLAlchemy is a SQL toolkit and Object Relational Mapper (ORM) that is flexible, extensible, and equally powerful.
SQLAlchemy can be seen as a database domain-specific language written in Python as it allows developers to write complex
SQL expressions without the need of writing raw SQL commands in many scenarios. SQLAlchemy is presented as two distinct
APIs, namely the Core and the ORM. The ORM builds on the Core and in many scenarios, developers can manage all database
services using the core without any ORM feature.

SQLAlchemy Core as the name implies is the foundation part of the system that provides database management and tooling.
It also provides a rich SQL Expression Language that allows for building queries by combining various constructs that
may include SQL functions, keywords, queries, or command. The Core library provides tools for managing connectivity to a
database, interacting with database queries and results, and programmatic construction of SQL statements.
The SQLAlchemy ORM provides object relational mapping support allowing user have access to Python model classes that are
mapped to database tables. The ORM extends Core features like SQL Expression Language and a session object to save
changes to the database.

We will explore some aspects of the Core API and see how to use this in managing database records for the mWallet
application.

#### 📚 Further Read – SQLAlchemy features

> SQLAlchemy boasts of several features that sets it apart from other ORMs written in Python. Some key features are:
> + Database operations can be performed without an ORM
> + SQLAlchemy is a matured, robust architecture with high performance
> + SQLAlchemy conforms with multiple relational databases’ requirements
> + The framework is non-opinionated SQL toolkit written fully in Python
> + It adopts the unit of work pattern which batches pending SQL operations (insert, update, delete etc.) into queues
    and flushes it at once. This makes if fully transactional and safe with high efficiency.
> + SQLAlchemy is modular and extensible
> For more information on the features of SQLAlchemy see [documentation here](https://www.sqlalchemy.org/features.html) .

### Connecting to the database

The tool for connecting to a relational database system is provided by the SQLAlchemy Core. We must first establish a
connection to the database before using SQLAlchemy Core to access it. An `Engine` class is used to set up and manage a
database connection and database specific dialect. To create an `Engine` entail passing a URL connection string to
sqlalchemy's `create_engine` function. It is important to note that though an `Engine` instance is returned by
the `create_engine` function, an actual database connection is not established until the `Engine.connect()`
or `Engine.execute()` methods are called either directly or through a `session` object associated with the `Engine`
instance. In effect, the `Engine` has a lazy initialization behaviour.

A database URL connection string follows comprises segments that specifies the database dialect, the driver,
datasource name or username and password.

{{<figure title="Figure 2" src="connection-url.png" caption="SQLAlchemy database connection to a sample Postgresql database system using Python driver [`postgresql`](https://pypi.org/project/psycopg2/)" >}}

In some instances, the sqlalchemy URL class can be used to generate a URL object that can be used in place of a
connection string. This is desirable where username or password fragment of a connection string may have forbidden
characters, for example, `@` or `-`.

{{<code lang="py3">}}
from sqlalchemy import create_engine
from sqlalchemy import URL

engine_a = create_engine("postgresql+psycopg2://fred:randompassword@localhost:5432/mWallet")

url_object = URL.create(
    "postgresql+psycopg2",
    username="fred",
    password="random-password"  # An unacceptable character in password. This will raise an error if using connection string
host = "localhost",
database = "mWallet",
)

engine_b = create_engine(url_object)

{{</code>}}

We initialized two engines, `engine_a` and `engine_b` using the `create_engine` function. In the first case, a string
parameter was used with `create_engine` while a `url_object` was used in the later case. Both `Engine` instances are
valid connections to the underlying database named mWallet. Instances of the `Engine` class (for example, `engine_a`
and `engine_b`) can be passed to a `Session` object to work with the SQL Expression Language or ORM. Also, note that
both `engine_a` and `engine_b` are defined as global variables.

🔍

> SQLAlchemy provides multiple methods in producing an Engine instance aside the create_engine function. Other functions
> are the engine_from_config and create_mock_engine.
> For more information on database connections, see
> the [SQLAlchemy documentation](https://docs.sqlalchemy.org/en/20/core/engines.html#engine-creation-api)

An instance of the `Engine` class could be defined as a global object to make it readily accessible to other modules or
packages. So, let’s do some housekeeping. We will define a `db` package in which all database services will be managed.
This package will access our connection string stored as an environment variable. It is a good practise to have database
connection string and similar credentials as environment variables using a `.env` file which is not directly accessible
in code. Also, ensure that `.env` file or other files used to store environment variables are not added to your code
repository or version control tool.

#### 📚 Further Reading - SQLAlchemy Engine

> {{<figure src="sqla_engine_arch.png" title="Figure 3" caption="SQLAlchemy engine architecture showing how the Engine references both a Dialect and a Pool object which provides DBAPI functions and database behaviour." >}}
> The SQLAlchemy Engine is the starting point of the all database operations defined in the SQLAlchemy DBAPI. Actual
> database operations and connection pool are managed by an Engine instance.
> + 📖 [Explaining how to work with SQLAlchemy Engine and Connections](https://docs.sqlalchemy.org/en/20/core/connections.html#basic-usage)
> + 📖 [SQLAlchemy Engine configuration](https://docs.sqlalchemy.org/en/20/core/engines.html)
> + 📖 [SQLAlchemy database dialect](https://docs.sqlalchemy.org/en/20/dialects/index.html)

In the mWallet application, all databse services for setting up connections, and defining database models are managed
through the `db` package. An `engine` instance is defined as a global variable which can be injected across various
functions when needed as only a single connection is needed per time for each user session.

[^engine_arch]: Figure of the SQLAlchemy engine architecture showing how the Engine references both a Dialect and a Pool object which provides DBAPI functions and database behaviour.

{{<code lang="py3">}}

import datetime as dt
from typing import Any, Type, Dict

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from bitmast_mwallet.settings import DB_CONNECTION_URL

engine = create_engine(DB_CONNECTION_URL, echo=True)
Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


def create_db():
    Base.metadata.create_all(bind=engine)


def drop_db():
    Base.metadata.drop_all(engine)


def get_session():
    return Session()

{{</code>}}

In our setup we introduced the `sessionmaker` function. This function is used to generate `Session` objects. In our
scenario, we will be using the same database configuration hence we can provide these configuration details as
parameters to the `sessionmaker`. Keyword parameters that are used for `Session.configure` can be passed to
the `sessionmaker` function to generate a `Session` with the set configuration. In our example, we pass three
parameters: `autocommit` which is a boolean specifying if database transactions should be automatically saved even if
the session `commit()` method is not called; `autoflush ` which is a boolean specifying if transactions should be
automatically flushed; `bind` variable which specifies what engine is used to manage the underlying database and by
implication, the engine to which all transactions will be performed.

We also defined a `Base` variable using the `declarative_base()` function. This function call will generate a base class
from which we can derive model classes that we can use to describe underlying database tables declaratively.
The `declarative_base` function can be called with keyword argument to further customise the target object used for
mapping models to database table. However, for our use case, the default values are suitable. All model classes derived
from our Base object will implicitly inherit the same mapper object that connects our model to the underlying database
tables.

We can further organize ORM models with similar properties to share the same `Base` class and other parents. This
approach will enable us customize table structure and behaviour using model classes or mixins that our final models will
inherit from. For instance, all tables with a primary key field represented as an integer can inherit from the same
parent mixin and tables with foreign key linked to a user record can inherit this feature from a mixin with this
characteristic. Tables which support a given search behaviour can inherit from a search mixin. SQLAlchemy provides us
tools to achieve this easily through the `declared_attr` and `declarative_mixin` decorators.

{{<code lang="py3">}}
import datetime as dt
from typing import Any, Type, Dict

import stringcase
from fastapi import Depends
from pydantic import BaseModel
from sqlalchemy import Column, String, Integer, Unicode, ForeignKey
from sqlalchemy import create_engine, DateTime, inspect
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.ext.declarative import declarative_base, declared_attr
from sqlalchemy.orm import sessionmaker, declarative_mixin, Mapped, mapped_column, relationship

from bitmast_mwallet.settings import DB_CONNECTION_URL
from util import generate_slug, current_timezone

engine = create_engine(DB_CONNECTION_URL, echo=True)
Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


def create_db():
    Base.metadata.create_all(bind=engine)


def drop_db():
    Base.metadata.drop_all(engine)


def get_session():
    return Session()


@declarative_mixin
class TableMixin:
    @classmethod
    @declared_attr.directive
    def __tablename__(cls):
        name = cls.__name__
        return stringcase.lowercase(stringcase.snakecase(name))

    def __int__(self, *args, **kwargs):
        for c in inspect(self).mapper.column_attrs:
            setattr(self, c.key, kwargs[c.key])

    def generate_pydantic_schema(self, model: Type[Base], transpose: Dict = None) -> Any:
        fields = self.as_dict()
        fields.update({'Config.arbitrary_types_allowed': True})
        fields.update({'__annotations__': self.__annotations__})

        if hasattr(model, '__name__'):
            name = model.__name__
        else:
            name = 'DynamicSchema'

        if transpose:
            # set annotations using the schema
            fields = {x: fields.get(y, None) for x, y in transpose.items() if y is not None}

        schema = type(name, bases=(BaseModel,), dict=fields)

        return schema


@declarative_mixin
class BaseTableMixin:
    uuid: Mapped[UUID] = mapped_column(UUID, unique=True)
    slug: Mapped[str] = mapped_column(String(128), index=True, unique=True, default=generate_slug)

    @classmethod
    def search(cls, session: Any = Depends(get_session), **kwargs):
        with session.begin():
            return session.query(cls).filter_by(**kwargs)

    def _asdict(self):
        return {c.key: getattr(self, c.key) for c in inspect(self).mapper.column_attrs}

    def __eq__(self, other):
        if hasattr(self, 'id') and hasattr(other, 'id'):
            return self.__name__ == other.__name__ and self.id == other.id


@declarative_mixin
class PrimaryKeyMixin:
    id: Mapped[int] = mapped_column(Integer, primary_key=True, index=True, unique=True)

    def __eq__(self, other):
        return self.__class__.__name__ == other.__class__.__name__ and self.id == other.id


@declarative_mixin
class UserIdMixin:
    usid = mapped_column(String(32), unique=True, index=True, nullable=False)
    user_id = mapped_column(Integer, ForeignKey('user.id'))

    @declared_attr
    @classmethod
    def user(cls):
        return relationship('User', cascade='all, delete')

{{</code>}}

We define a `TableMixin`, `BaseTableMixin`, `PrimaryKeyMixin`, and `UserIdMixin`. These classes define different
features we may want to pass unto our model classes. The `TableMixin` defines a `__tablename__` attribute which converts
our class name to a snakecase (snake_case) string value. This parameter is used to set the actual table name created in
our database during migration. The `BaseTableMixin` provides three methods that allows us to search a table using
keyword arguments provided as a dictionary; an `_asdict` method that parse our ORM instance to Python dictionary having
attributes as keys and their respective value as values; an equality check method achieved by overridden the
default `__eq__` method.

🔍

> Overriding special or dunder (double underscore) methods in Python should be done with caution. These methods are
> often associated with the root object or a parent metaclass. Only when one is very sure of the desired outcome of
> overridden such method should such be done. The example in this book is simply for demonstration purposes and not a
> recommendation for the reader.

The `PrimaryKeyMixin` defines an id attribute that is used as primary key for any model inheriting from this class.
SQLAlchemy supports various database dialects and features such as composite primary keys. Classes inheriting
from `PrimaryKeyMixin` can define other primary key fields which will be converted to a composite primary key upon
migration. The `UserIdMixin` uses the `declared_attr` decorator to return a relationship object which links the derived
models of this class to a `User` model via the `user_id` attribute. In effect, the resulting table will have a `user_id`
that references the user table upon migration.

These classes can be used to define various ORM classes declaratively.

{{<code lang="py3">}}
from typing import Any
from enum import Enum

from sqlalchemy import Enum as DbEnum, Numeric, String, LargeBinary, ForeignKey, Integer, Text, SmallInteger, Boolean,
    UniqueConstraint
from sqlalchemy.dialects.postgresql import JSON
from sqlalchemy.orm import mapped_column, Mapped, relationship

from db import Base, BaseTableMixin, UserIdMixin, TableMixin, PrimaryKeyMixin, TimestampMixin
from util import generate_transaction_ref


class OrderType(Enum):
    OPEN = 0
    CLOSED = -1
    SATISFIED = 1
    PARTIAL = 2


class AssetType(Enum):
    BITCOIN = 2
    DASH = 3
    DOGECOIN = 5
    LITECOIN = 7
    NAIRA = -1


class Order(TableMixin, PrimaryKeyMixin, UserIdMixin, BaseTableMixin, Base):
    order_type: Mapped[OrderType] = mapped_column(DbEnum(OrderType))
    asset: Mapped[AssetType] = mapped_column(DbEnum(AssetType))
    start: Mapped[float] = mapped_column(Numeric)
    expected_end: Mapped[float] = mapped_column(Numeric)
    memo: Mapped[str] = mapped_column(String(128))


class TransactionKey(TableMixin, UserIdMixin, PrimaryKeyMixin, BaseTableMixin, Base):
    signature: Mapped[bytes] = mapped_column(LargeBinary)
    pubkey: Mapped[bytes] = mapped_column(LargeBinary)
    start: Mapped[float] = mapped_column(Numeric)
    end: Mapped[float] = mapped_column(Numeric)


class Transaction(TableMixin, PrimaryKeyMixin, BaseTableMixin, Base):
    order_id: Mapped[int] = mapped_column(Integer, ForeignKey('order.id'))
    order: Mapped[Order] = mapped_column('Order')
    tranx_ref: Mapped[str] = mapped_column(String(256), default=generate_transaction_ref)
    tranx_id: Mapped[str] = mapped_column(String(256), nullable=False)
    inputs: Mapped[str] = mapped_column(Text)
    outputs: Mapped[str] = mapped_column(Text)
    confirmations: Mapped[int] = mapped_column(SmallInteger)
    pubkey: Mapped[bytes] = mapped_column(LargeBinary)
    meta: Mapped[Any] = mapped_column(JSON)

{{</code>}}

We define an `Order` and `Transaction` model which both extends earlier defined mixins. Attributes of these models are
defined with annotated type hints. For example, the asset attribute of `Order` has a data type of `AssetType` which is
an enumeration of cryptocurrencies. The type hint is declared using the `Mapped` class as specified in SQLAlchemy 2.0.x.
The `Mapped` class provides information for type checkers like mypy so that ORM attributes managed by the base mapper
have appropriate type. The column type associated with each model attribute is declared using the `mapped_column`
function. Earlier version of SQLAlchemy (1.4 or earlier)  uses the `Column` class to declare table columns as attributes
of the model. The `mapped_column` still returns a `Column` object and takes keyword arguments that is used in `Column`
initialization.

We can expand further on our code to define other relevant classes that meets our minimum viable product. Once all our
classes are in place, we can migrate our models to actual database tables. A common migration tool used with SQLAlchmey
is the `alembic` package developed by the SQLAlchemy team. This can be installed using `pip` command. A detail tutorial
on migration is beyond the scope of this book. However, readers can visit the official website
of [alembic](https://alembic.sqlalchemy.org/en/latest/) for more information on using alembic.

#### 📚 Further Read - SQLAlchemy alembic

> Alembic uses SQLAlchemy as an underlying engine to manage database changes and other DDL operations like create,
> alter, truncate, and drop table.
> For more information on database connections, see the alembic documentation and tutorial
> + 📖 [Alembic documentation](https://alembic.sqlalchemy.org/en/latest/index.html)
> + 📖 [Alembic tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)

### CRUD operations - Eloquent SQLAlchemy queries

We have defined a couple of models using SQLAlchemy ORM. We now have a Python bridge with our database and can send and
retrieve data from our database using this bridge. In most web applications, we will be performing operations such as
create data, retrieve data, update data, and delete data – CRUD operations. SQLAlchemy Core and ORM provides us with
expressive query language by which we can perform SQL Data Manipulation Language (DML) with commands such as INSERT,
UPDATE, DELETE; Transaction Control Language (TCL) with commands such as COMMIT; Data Query Language with SELECT command
and lots more. We can define functions to execute CRUD operations and other operations across multiple models as needed
by our application. For example, we can have a function that will enable us to create arbitrary records of any model
with a set of values.

{{<code lang="py3">}}

from enum import IntEnum
from typing import TypeVar, Type, Mapping, Union, Any, Dict, Sequence, Tuple, List
from uuid import UUID

from pydantic import BaseModel as PydanticModel
from sqlalchemy import inspect, insert, update
from sqlalchemy.engine import Result, Row
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import Session, Query

from bitmast_mwallet import settings

Base = declarative_base()

ModelT = TypeVar("ModelT", bound=Base)
Queryset = TypeVar("Queryset", bound=Query)
QueryResult = TypeVar("QueryResult", bound=Result)
ResultRow = TypeVar("ResultRow", bound=Row)


class Operation(IntEnum):
    CREATE = 2
    READ = 3
    UPDATE = 5
    DELETE = 7
    LIST = 11


def model_retrieve(model_class: Type[ModelT], identifier: Union[int, str, UUID], skip: int = 0, limit: int = 0,
                   filter_by: Dict = None, exclude: List | Tuple = None) -> Any:
    with Session(settings.DB_ENGINE) as session:
        if isinstance(identifier, UUID):
            identifier = str(identifier)

        if filter_by:
            queryset = session.query(model_class).filter(
                model_class.slug == identifier).filter_by(**filter_by)
        else:
            queryset = session.query(model_class).filter(
                model_class.slug == identifier)

        result = queryset_range(queryset, skip, limit)
        return parse_to_dict(result, exclude=exclude)


def model_list(model_class: Type[ModelT], skip: int = 0, limit: int = 0, filter_by: Dict = None,
               exclude: List | Tuple = None):
    with Session(settings.DB_ENGINE) as session:
        queryset = session.query(model_class)
        if queryset:
            if filter_by:
                qs = queryset_range(queryset, skip, limit, as_queryset=True)
                rs = qs.filter_by(**filter_by).all()
            else:
                rs = queryset_range(queryset, skip, limit)
            return parse_to_dict(rs, exclude=exclude)


def model_bulk_create(instance: Union[List[Type[ModelT]], Tuple[Type[ModelT]]] = None) -> Any:
    with Session(settings.DB_ENGINE) as session:
        if isinstance(instance, (List, Tuple)):
            session.add_all(instance)
            session.commit()
            return parse_to_dict(instance)


def model_create(instance: Union[Type[ModelT], List[Type[ModelT]], Tuple[Type[ModelT]]] = None, data: Mapping = None,
                 model: Type[ModelT] = None, exclude: List | Tuple = None) -> Any:
    with Session(settings.DB_ENGINE) as session:
        if instance:
            session.add(instance)
        elif data and model:
            instance = model(**data)
            session.add(instance)
        session.flush()
        session.refresh(instance, ('id',))
        result = parse_to_dict(instance, exclude=exclude)
        session.commit()
        return result


def model_update(model_class: Type[ModelT], data: Mapping = None, identifier: Union[int, str, UUID] = None,
                 filter_by: Dict = None, exclude: List | Tuple = None) -> Tuple[Sequence, int]:
    with Session(settings.DB_ENGINE) as session:

        if identifier and filter_by:
            queryset = session.query(model_class).
                filter(model_class.slug == identifier).filter_by(**filter_by)
            if queryset:
                updated = queryset.update(values=data, synchronize_session='fetch')
                rs = queryset.all()
                updates = parse_to_dict(rs, exclude=exclude)
                return updates, updated
        elif identifier and not filter_by:
            queryset = session.query(model_class).
                filter(model_class.slug == identifier)
            if queryset:
                updated = queryset.update(values=data, synchronize_session='fetch')
                rs = queryset.all()
                updates = parse_to_dict(rs, exclude=exclude)
                return updates, updated
        elif filter_by and not identifier:
            queryset = session.query(model_class).filter_by(**filter_by)
            if queryset:
                updated = queryset.update(values=data, synchronize_session='fetch')
                rs = queryset.all()
                updates = parse_to_dict(rs, exclude=exclude)
                return updates, updated
        session.commit()


def model_destroy(model_class: Type[ModelT], identifier: Union[str, int, UUID] = None, filter_by: Dict = None) ->
        Union[None, int]:
    with Session(settings.DB_ENGINE) as session:
        if identifier and filter_by:
            queryset = session.query(model_class).
                filter(model_class.slug == identifier).filter_by(**filter_by)
            if queryset:
                deleted = queryset.delete(synchronize_session='fetch')
                return deleted
        elif identifier and not filter_by:
            queryset = session.query(model_class).filter(model_class.slug == identifier)
            if queryset:
                deleted = queryset.delete(synchronize_session='fetch')
                return deleted
        elif filter_by and not identifier:
            queryset = session.query(model_class).filter_by(**filter_by)
            if queryset:
                deleted = queryset.delete(synchronize_session='fetch')
                return deleted
        session.commit()


def crud_operation(model_class: Type[ModelT], operation: Operation, data: Dict = None,
                   identifier: int | str | UUID = None, skip: int = 0, limit: int = None,
                   filter_by: Dict = None) -> Any:
    if operation in [2, 'create', Operation.CREATE]:
        return model_create(model=model_class, data=data)
    elif operation in [3, 'read', Operation.READ]:
        return model_retrieve(model_class=model_class, identifier=identifier, skip=skip, limit=limit,
                              filter_by=filter_by)
    elif operation in [5, 'update', Operation.UPDATE]:
        return model_update(model_class=model_class, identifier=identifier, data=data, filter_by=filter_by)
    elif operation in [7, 'delete', Operation.DELETE]:
        return model_destroy(model_class=model_class, identifier=identifier, filter_by=filter_by)
    elif operation in [11, 'list', Operation.LIST]:
        return model_list(model_class, skip=skip, limit=limit, filter_by=filter_by)

{{</code>}}

We define the `model_create`, `model_retrieve`, `model_update`, `model_destroy`, and `crud_operation` function.
The `crud_operation` function relies on the `model_create` to save new records of the specified model using the values
passed as data parameter. The `model_retrieve` function retrieves any arbitrary record using a specified identifier
which can be a unique attribute of the model, for example, slug field, primary key, or a uuid field. Where
the `identifier` is not provided, a set of attributes (keys) with their corresponding values can be used. These
key-value pairs should be passed in as a dictionary through the `filter_by` parameter. The `model_update` function
modifies existing records of the model specified using the provided data. This function is a bulk operation by default
as more than one result may be matching the resulting query and hence multiple updates can be done at once. In a
scenario where only, a unique identifier is used, then a single record is expected to be modified or updated. Similar to
the `model_retrieve`, the `filter_by` parameter can be used to specify attributes by which filtering can be performed.
Each attribute should be passed as a dictionary key with their corresponding values. The `model_destroy` function is
performs SQL queries like the `model_update` but it deletes matching records unlike the `model_update` that modifies
matching records. Both the `model_update` and `model_destroy` returns a tuple of records affected and the count of
values updated or deleted. Both `model_update` and `model_destroy` should be used cautiously as affected records will be
modified or deleted once the Session object `commit()` function is called or the context manager terminates – depending
on the Session configuration like `autocommit` being set to `True`.

The `filter_by` parameter is used in the query expression to perform an equality check by default. This is a current
limitation of the defined `crud_operation` function. A more powerful definition can make provision for specify
conditional checks like using the OR, IN, NOT, LIKE and other SQL functions. Where such is desirable, defining a
function that captures the specific business case can be used. However, in many situations, it is sufficient to retrieve
or modify data using exact values, datatypes, and field names as portrayed in the `crud_operation`.

The `skip` and `limit` parameters of `model_retrieve`, `model_update` and `model_destroy` are used to specify the start
and end indexes of the returned query results where more than one record matches the query criteria.

In the design of the `crud_operation`, we have used a set of flags to indicate what operation to perform. We also have
used a context manager derived from a Session object in each of the composed functions
of `crud_operation` (`model_create`, `mode_retrieve`, `model_destroy`, `model_update`). This is an alternative approach
to using a `sessionmaker` as defined earlier. Another approach entails injecting a session parameter into a function
using Dependency Injection. The FastAPI provides the `Depends` class towards that purpose. If using this approach, we
simply pass a `session` into the desired code as a parameter with default value returned from `Depends` class.
The `Depends` class can take a callable or other function construct that returns the desired data or object to pass as a
parameter. This approach is common amongst FastAPI web applications, and it makes for easier to manage code as functions
are loosely coupled.

With the `crud_operation` in place, we can focus on defining application centric queries for analysis and other business
requirements. For example, we can define a query function to retrieve all transactions within a set period of start and
end dates.

{{<code lang="py3">}}

import datetime as dt
import json

from typing import Dict
from fastapi import Depends
from sqlalchemy import or_, insert
from sqlalchemy.orm import Session

from db import get_session
from db.adapters import queryset_range
from models import AppNotice, User, Organization
from models.administration import notice_recipient_association


def list_notice_for_a_user(usid: str, start: dt.datetime, end: dt.datetime, skip, limit,
                           s: Session = Depends(get_session)):
    with s.begin():
        user_query = s.query(User.id).filter(or_(User.slug == usid, User.rsid == usid)).scalar_subquery()
        if start and not end:
            queryset = s.query(AppNotice).join(User).filter(AppNotice.start_at >= start, User.id == user_query)
        elif end and not start:
            queryset = s.query(AppNotice).join(User).filter(AppNotice.end_at <= end, User.id == user_query)
        else:
            queryset = s.query(AppNotice).join(User).filter(User.id == user_query)

        return queryset_range(queryset, skip, limit)

{{</code>}}

We define a hypothetical `list_notice_for_a_user` function which retrieves a list of notices for the specified user. The
session object is passed to the function as a parameter `s` using the `Depends` class. The `Depends` class takes a
function `get_session` defined in our `db` package which returns an instance of `Session` class.

### Threading database operations

In a web application, HTTP requests are sent to a server which may return a response after some operations. Certain
times data may be read from a file, a database or third-party system. If the operation is synchronous, no other
operation will be performed till the current request has been handled. When a synchronous operation is ongoing, a client
may simply wait as the system can only respond after executing the current task. This may not be desirable in many
scenarios. Take an example where we are to retrieve all transaction of a blockchain on a given network. Having millions
of transactions, means we may wait significant time before all transactions are loaded into server memory and then
passed unto our client, the browser. We could paginate our responses so we lazy load data and set a few results per page
instead of reading all million transactions at once. This could improve the user’s perceived sense of speed of the
application. The skip and limit parameter introduced in our `crud_operation` can help us define a variable length of
data to fetch per time. We can also set these parameters to `None` or `0` to retrieve all available results at once.

Despite this improvement, we are still waiting for our blockchain network or a database operation to execute and give
responses before rendering results as pages. We can consider another scenario in which our database operation can run on
a different process or thread? Will there be an improved service? Furthermore, Python language provides several
mechanisms for iterating a sequence and mechanism for parallel operations or concurrency. Can these be applied to
database operations for web frameworks? The FastAPI framework is an async framework so asynchronous functions or
coroutines can be used with the FastAPI application instance. SQLAlchemy 2.0 also support AsyncEngine and async session
object allowing for asynchronous query of database, that is, queries can run concurrently. Depending on application
requirements, these options can be explored to improve the quality and performance of code. A word of warning should be
given at this point. Async programming is complex and not suitable in every instance. Furthermore, certain operations
simply cannot be executed asynchronously. We need to ensure that the right solution is being used for the right problem
whether we are dealing with asynchronous framework or simply printing “Hello World” to a terminal. We will use a
combination of Python Coroutine, Future, and Generator to improve on our database services. This approach is being
explored as it enables us to still use SQLAlchemy ORM Session and not switch to asynchronous Session which are not
widely used across web applications at the time of writing.

To achieve our goal of delegating database operations to a separate thread, we will define a `delegate_task` function
that will be responsible for managing our threaded operations within a context manager. Python provides a high-level
means for concurrent operations using the `ThreadPoolExecutor` or `ProcessPoolExecutor` context manager of the
concurrent.futures package. We will use the `ThreadPoolExecutor` to initialize our database sessions on a separate
thread as this operation is IO bound and not CPU bound.

{{<code lang="py3">}}
from typing import Generator
from concurrent.futures import ThreadPoolExecutor


def delegate_task(task: Callable, workers: int = 2, use_max: bool = False, args: Sequence = None,
                  **kwargs) -> Generator:
    if workers < 0:
        workers = 2
    if use_max:
        workers = 5
    with ThreadPoolExecutor(max_workers=workers) as ex:
        if args and kwargs:
            future = ex.submit(task, *args, **kwargs)
        elif args and not kwargs:
            future = ex.submit(task, *args)
        elif kwargs and not args:
            future = ex.submit(task, **kwargs)
        else:
            future = ex.submit(task)
        yield future.result()


{{</code>}}

The defined `delegate_task` function takes a callable named `task` and specifies
parameters: `workers`, `use_max`, `args`, and `kwargs`. The `workers` parameter is an integer value indicating the
number of threads to manage using the `ThreadPoolExecutor`, `use_max` boolean parameter specifies if the default maximum
number of threads defined for the application should be used instead. The `args` and `kwargs` are passed to the
callable `task` which is run within the `ThreadPoolExecutor` context manager via the `Executor` instance, `ex`. If the
callable is ran successfully, a `future` object is returned via the ex.submit() function call. We can then wait for the
outcome of the `future` object by calling its result method. Using this approach, we can set up a database session,
establish connection with the database on a separate thread, and retrieve the database connection by
calling `future.result()`. Notice the use of the `yield` reserved keyword to pass `future.result()` as a `Generator`
object. This implies the output of calling `future.result()` is wrapped as a `Generator` object and then passed to the
caller. Generators are good for iterating values and very efficient in memory usage. This way, we can generate database
connection efficiently on separate threads and lazily retrieve query results when needed. Let us also update
our `get_session` function of `db` package to use concurrent programming.

{{<code lang="py3">}}
import datetime as dt
import types
import uuid
from decimal import Decimal
from enum import Enum
from typing import Dict, TypeVar, List, Tuple, Generator

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session

from bitmast_mwallet.settings import DB_CONNECTION_URL
from util import delegate_task

engine = create_engine(DB_CONNECTION_URL, echo=True)
Base = declarative_base()

ModelT = TypeVar('ModelT', bound=Base)


def create_db():
    Base.metadata.create_all(bind=engine)


def drop_db():
    Base.metadata.drop_all(engine)


def get_session():
    session = delegate_task(sessionmaker, workers=2, autocommit=False, autoflush=False, bind=engine)
    if isinstance(session, Generator):
        result = tuple(session)
        return result[0]()

{{</code>}}

We update the get_session function to return an instance of `Session`. However, this instance is retrieved from
a `Generator` object returned by the `delegate_task` function. We validate that a `Generator` object is returned and
then consume all values in the Generator instance by passing it to a tuple. We can then return the first object from our
tuple which is expected to be a Session as that is the returned value from `sessionmarker`. With this in place all
queries from our application can be run by passing our session object generated on a separate thread.

🔍

> An alternative approach of retrieving a session object from the output of `get_session` is to call the `next()`
> function with a `Generator` as its only parameter. This call is similar to retrieving the next value of a `Generator` by
> calling the `__next__` method of the object. If no more values are available in the Generator, a `StopIteration` error
> is raised.

{{<code lang="py3">}}
# Alteranate approach using Python next() builtin function 

try:
    gen_session = delegate_task(sessionmaker, workers=2, autocommit=False, autoflush=False, bind=engine)
    session = next(gen_session)
    return session
except StopIteration:
    pass

{{</code>}}

### Database transactions

A database transaction may comprise multiple SQL operations that are treated as a single unit of work. A major
characteristic of a database transaction is that it cannot leave the database in an inconsistent state. Database
transactions have the following characteristic:

1. Atomicity – each SQL statement in a transaction must be completely executed or not executed at all. This requires
   that database queries are saved to persistent memory and does not leave the database in an inconsistent state. Where
   a query fails, the entire transaction will be rolled back to ensure a clearly defined state is maintained.
2. Consistent – following the proper execution of queries, database must change from one known state to another. Each
   state change must be valid.
3. Isolation – an unsaved or uncommitted SQL query should not affect execution of another query or concurrent
   transaction.
4. Durability – a committed transaction must change the state of the database.
   SQLAlchemy queries are executed as database transactions hence failed queries are rolled back by default. Users can
   also compose their queries using different constructs to obtain a desired SQL expression or result. Much flexibility
   in drafting SQL queries is achieved via the SQLAlchemy Core Query Expression language. Furthermore, query results can
   be managed using the sqlalchemy.engine.Result class and other datatypes
   like `CursorResult`, `MappedResult`, `Row`, `RowMapping` etc. More details on various result types can be seen in
   the [SQLAlchemy documentation](https://docs.sqlalchemy.org/en/20/core/connections.html#result-set-api) .

While the result from a query may be parsed as tuple, dictionary or even model instances, in some cases, it may be
desirable to lazily consume query results. This will enable us to effectively use limited system resource. We will
consider our previous example of retrieving all transactions from a given blockchain network. Earlier we had considered
using pagination to improve on the response of our system. Now let’s adopt similar approach of using threads to manage
query results as we did with initializing database sessions. Similar to the `delegate_task` function, we will define a
decorator to serve as a wrapper for query functions that will yield a `Generator` which we can then iterate or consume
at once.

{{<code lang="py3">}}
from concurrent.futures import ThreadPoolExecutor


def thread_runner(workers: int, use_max: bool = False):
    if workers < 0:
        workers = 2 or settings.DEFAULT_THREAD_WORKERS
    if use_max:
        workers = 5

    def decorate(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            try:
                with ThreadPoolExecutor(max_workers=workers) as ex:
                    future = ex.submit(func, *args, **kwargs)
                    gen = future.result()  # the passed callable must return a Generator object
                    if isinstance(gen, Generator):
                        return tuple(gen)
                    else:
                        raise TypeError('Expected type <Generator> but got %s' % type(gen))

            except Exception:
                raise

        return wrapper

    return decorate

{{</code>}}

We used the `ThreadPoolExecutor` to execute the query function define as `func` within a context manager. The `func`
must return a `Generator` object else the decorator will raise a `TypeError`. We may consume all results from
the `Generator` at once by passing it to the tuple class or we can return the `Generator` object for the other functions
to iterate or consume. In our example, we simply consume `Generator` data and load this into memory. With our decorator
in place, we can now modify the behaviour of our functions for running queries making them execute queries on a separate
thread from the main thread running our application.

The use of separate thread for database queries is not a general recommendation for every possible business case. It was
primarily highlighted here to show readers techniques in which long-running processes in their web application can be
improved using Python Generator and concurrency. Also, this approach was explored in the domain of cryptocurrency
payment as a means of reducing latency in placing or satisfying trades. Assuming we have a high server hit rate for our
remote database and blockchain API server, we will be able to send many more requests in a shorter time by using
concurrency when compared to sending requests sequentially. The reader can still explore other alternatives in
maximizing response, for example, the use of cache for retrieving already confirmed blockchain transactions.

#### 📚 Further Reading - Python database drivers

> In this Chapter we have explored the use of SQLAlchemy to manage our database system. The SQLAlchemy is a good ORM for
> relational database and at the time of writing it does not support NoSQL databases like MongoDB. There are Python
> plugins for NoSQL databases with can be integrated into FastAPI applications. FastAPI approach of flexibility over
> convention allows developers to use existing Python tools to meet their business needs. Below are some plugins for
> managing some popular database systems which can be integrated into a FastAPI application.
> + 📖 [DataStax](https://docs.datastax.com/en/developer/python-driver/3.24/) for Cassandra database
> + 📖 Python [MongoDB](https://www.mongodb.com/docs/drivers/pymongo/) driver for synchronous applications
> + 📖 Python [MongoDB](https://www.mongodb.com/docs/drivers/motor/) driver for asynchronous applications
> + 📖 Python driver [redis-py](https://github.com/redis/redis-py) for Redis database 

