+++
title = 'Authorization and Authentication'
date = 2023-11-05T22:04:30+01:00
weight = 9
draft = true
+++


Authentication and access control are an important aspect of many applications. These layers of security should not be ignored or treated as secondary but as a primary aspect of the system. In some cases, security and usability may require further deliberation to maximize benefits for the user. Security guidelines by consortiums like Open Web Application Security Project (OWASP) are helpful across various mobile and web applications. Other groups like Symposium on Usable Privacy and Security (SOUPS) focuses on making usable security available for users. In this Chapter, we will explore security in the FastAPI framework and design an approach of dynamically loading security settings for our mWallet application.

🎓
> Some recommendations for improved security for online users adapted from research publish by SOUPS include:

✅ Install Software Updates

✅ Use Unique Passwords

✅ Use Two-Factor Authentication

✅ Use Strong Passwords

✅ Use a Password Manager

📖 [SOUPS Conference paper on security] (https://www.usenix.org/system/files/conference/soups2015/soups15-paper-ion.pdf)

📖 [Recommendation on security](https://arstechnica.com/information-technology/2015/07/what-amateurs-can-learn-from-security-pros-about-staying-safe-online/) 

OWASP discussed top ten security risks to consider for improving code security (https://owasp.org/www-project-top-ten/).  These include:
1.	Broken Access Control
2.	Cryptographic failures
3.	Injection
4.	Insecure design
5.	Security misconfiguration
6.	Vulnerable and outdated components
7.	Identification and authentication failure
8.	Software and data integrity failure
9.	Security logging and monitoring failure
10.	Server-side Request Forgery (SSRF)

Authentication and Identification is a major security risk. Authentication are measures put in place to ensure that users are who they said they are. This is different from Access Control or Authorization which determines what resource or information a user can access or operate on. Privacy on the other hand is concerned with measures to control what aspect of a user’s data is visible to third parties and the measures available to users to control access to their personal data. In managing authentication, multiple security steps should be taken to reduce the possibility of an attacker. Some guidelines following recommendations from OWASP and SOUPS are given below. 

+ Reduce the use of leaked or published usernames and password. For example, the application should not permit default, weak, or well-known passwords, such as "Password1" or "admin/admin".
+ Adopt measures to prevent or limit brute force or other automated attacks.
+ Password and credential recovery processes should not leak user personal information or assume only "knowledge-based answers" (for example, name of pet) which cannot be made safe.
+ Where possible, use MFA along with username and password login. If only password is used, then encourage the use of longer passwords of ten characters or more.
+ Do not expose session identifier in URL.
+ Expire or invalidate session, JWT token, or similar credentials after user logout or been idle for a period.

A detailed security system cannot be effectively covered in a book like this. We will attempt a few security measures and explain how these were implemented in the mWallet application using FastAPI. The following security features will be adopted:

+ A restricted username and password list will be used to prevent the use of leaked or published username and password combination. This list is adopted from the github repository by Daniel Miessler ( https://github.com/danielmiessler/SecLists) and recommended by OWASP on implementing Digital Identity (https://owasp.org/www-project-proactive-controls/v3/en/c6-digital-identity). 
+ User group linked to named scope will be used to identify what API endpoint will be accessible to the various categories of users. This approach was adopted to simplify access based on user group which may be linked to one or more scopes. In effect, the scope property of our JWT will also match scope variables assigned to a given user group. A detailed alternative may define a Scope model and map one or more scopes to any combination of user Group instances. 
+	Financial transaction limits will used to limit the number of, and value of transactions performed by each tier of users. Detail implementation of this is beyond the scope of this book.
+	User authentication will adopt username and password along with One Time Passcode (OTP). Each failed login leads to an increasing wait time and ultimately blacklisting of IP address or username or both.

With this in view let us explore authentication in FastAPI.

🎓

> Recommended security guidelines from OWASP were used to define the security policy of the mWallet application and the security topics in this book. Further details can be found at the OWASP official website. 

📖 [Identification and authentication failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/) 

📖 [Web application security testing](  https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/04-Authentication_Testing/README) 

📖 [Implement Digital Identity](https://owasp.org/www-project-proactive-controls/v3/en/c6-digital-identity)

## Authentication

Authentication is an integral part of any web application. Various schemes are available, and some techniques are shared across web frameworks. For example, Django framework has built in user Authentication unlike FastAPI that developers implement authentication scheme of their choice, delegate authentication to external service, or rely on authorization specifications like OAuth2 via the `fastapi.security package`. Furthermore, although the FastAPI is built on the Starlette framework, the mechanism for authenticating and granting access to an endpoint in these frameworks is different. While FastAPI uses dependency injection to inject authentication code in each path operation, Starlette adopts an `AuthenticationMiddleware` by which all endpoints or selected endpoints can enforce authentication. The FastAPI framework supports OAuth2 and uses this for authorising user’s access to a resource. OAuth2 is industry standard authorisation for web API. A common setup is to have application servers delegate authentication service to a server such as Google, Facebook to perform authentication of a user, and then grant limited access to a resource based on the authentication outcome. In effect the application server delegates the service of identifying the user based on some credentials to a trusted third party that implements the OAuth2 specification. The application server can also serve as an authorisation and resource server while still providing access to third-party authentication using OAuth2 specifications. FastAPI provides a simplified and yet very effective means of authenticating users by implementing `OAuth2` class that extends `SecurityBase` class. The `OAuth2` class enables developers specify what endpoints requires authentication and authorize access based on an API token generated after duly validating a username and password. This approach is managed via the `OAuth2PasswordBearer`. The `OAuth2PasswordBearer` extends the `OAuth2` class and handles generating authorisation token using a username and password credentials. 

~~~python

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from bitmast_mwallet import settings


app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl=settings.AUTH_TOKEN_URL)

@app.get("/access/")
async def user_access(token:str = Depends(oauth2_scheme)):
    return {"token": token}

~~~

By initializing an instance of the `OAuth2PasswordBearer` class, we can specify any endpoint needing users to have authorized access through dependency injection. In our example, we defined a token URL assigned to `tokenUrl` parameter by which users can obtain or refresh authorisation tokens. The path operation function will take a valid token as string returned by calling `Depends(oauth2_scheme)` which injects the result of calling the instance of `OAuthPasswordBearer`, `oauth2_scheme`. The `oauth2_scheme` as an instance of `OAuth2PasswordBearer` is also callable. Calling this function will ensure a security scheme is attached to the endpoint and returns a string representing a token that can be used with Authorization Bearer in the client’s request header. Where the application also implements authentication, the username and password values posted to `user_access` endpoint can be checked against a record (usually, retrieved from a database) to approve or reject access. 

This scenario of using a username and password to obtain a token is defined in the OAuth2 specification as password flow. It is sufficient for some businesses cases. There are other flows defined in the OAuth2 specification which developers can adopt depending on their use case. FastAPI provides tools by which the basic OAuth2 password and other flows can be integrated into an application. For our project, we will define an `AuthHandler` class that uses the `HTTPBearer` class for authentication. With this class we receive login requests and generate a valid JWT token for the user. We will also use the `AuthHandler` class to encrypt user password using a strong cryptographic hashing algorithm. This is a recommended practise as passwords should not be saved as plain text. 

~~~python

import base64
import logging
import re
from datetime import timedelta
from typing import Any, Dict, List, Tuple, TypeVar, Generator

import stringcase
from fastapi import HTTPException, status, Body, Request, Security, Query, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials, SecurityScopes
from jose import jwt
from passlib.context import CryptContext
from sqlalchemy import create_engine, or_, insert
from sqlalchemy.orm import Session

from app_cryptography import key_hashing, verify_key_hashing
from bitmast_mwallet import settings
from db import get_session
from messages import error_messages
from models.administration import User, Group, user_group_association, UserType
from schemas.administration import LoginData
from signals.authentication import generate_otp_signal, sign_in_signal, sign_up_signal, password_reset_signal
from signals.event_bus import EventBus
from util import current_timezone, current_timestamp, thread_runner

engine = create_engine(settings.DB_CONNECTION_URL, echo=True)

logger = logging.getLogger(__name__)

credentials_exception = HTTPException(
    status_code=status.HTTP_401_UNAUTHORIZED,
    detail="Could not validate credentials",
    headers={"WWW-Authenticate": "Bearer"},
)

ALGORITHM = settings.AUTHENTICATION_ALGORITHM
ACCESS_TOKEN_EXPIRE_MINUTES = settings.ACCESS_TOKEN_EXPIRE_MINUTES

class AuthHandler:
    security = HTTPBearer()
    pwd_context = CryptContext(schemes=settings.PASSWORD_HASH_SCHEMES, deprecated='auto')
    secret = settings.SECRET_KEY
    scopes = None

    def __init__(self, scopes: List | Dict = None):
        if scopes and isinstance(scopes, List):
            self.scopes = [x for x in scopes if isinstance(x, str)]
        elif scopes and isinstance(scopes, Dict):
            self.scopes = [x for x, _ in scopes.items()]
        else:
            self.scopes = UserType.get_scopes()

    def get_password_hash(self, password):
        return self.pwd_context.hash(password)

    def verify_password(self, plain_password, hashed_password):
        return self.pwd_context.verify(plain_password, hashed_password)

    def encode_token(self, user, scopes: List = None, nonce: int | str = None) -> str:
        start = current_timezone()
        end = start + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
        payload = {
            'exp': end,
            'iat': start,
            'sub': user,
        }

        if isinstance(nonce, (int, float, str)):
            nonce = str(nonce)
            payload.update(cnonce=nonce)

        if scopes:
            s = get_session()
            with s.begin():
                if s.query(Group.scope).join(User).filter(Group.scope.in_(scopes), User.slug == user).all():
                    payload.update(scopes=scopes)
                else:
                    payload.update(scopes=[])
                s.commit()

        return jwt.encode(payload, self.secret, algorithm=settings.AUTHENTICATION_ALGORITHM)

    def decode_token(self, token, flag: int = 0) -> str | Dict:
        try:
            payload = jwt.decode(token, self.secret, algorithms=[settings.AUTHENTICATION_ALGORITHM])
            if flag == 0:
                return payload.get('sub')
            elif flag == 1:
                return payload
        except jwt.ExpiredSignatureError:
            credentials_exception.detail = 'Signature has expired'
            raise credentials_exception
        except jwt.JWTError:
            credentials_exception.headers.update({'WWW-Authenticate': 'Bearer'})

    def auth_wrapper(self, authentication_handler: HTTPAuthorizationCredentials = Security(security, scopes=scopes)):
        return self.decode_token(authentication_handler.credentials)

    def acl_wrapper(self, authentication_handler: HTTPAuthorizationCredentials = Security(security, scopes=scopes)):
        return self.decode_token(authentication_handler.credentials, flag=1)

~~~

We defined an `AuthHandler` class to manage selected services relating to user authentication. As an example, we will need to encrypt all user password before storing them to a database. The `AuthHandler` class will provide the needed hashing method to secure our password records. A suitable hashing algorithm, preferably of a fixed length and slow speed, should be selected. This will help reduce brute force and timing attacks. Python programming language supports the BCRYPT algorithm, and this is use for password hashing due to its slow speed even on fast hardware. In our implementation, we used the `passlib.context.CryptContext` class to initialize a context object from which we can perform password hashing and password hash verification. The context object will internally select a matching algorithm for hashing based on the configuration in our settings. `PASSWORD_HASH_SCHEMES` which returns a list of approved algorithms for hashing. The `AuthHandler` `get_password_hash` method is used to generate hashes for a given password using our declared context object as explained earlier. To authenticate a user, we will need to ensure that password entered during login can be verified against an existing hash assigned to the user. We will use the `AuthHandler` `verify_password` method to perform this operation. This method takes a plain password entered by a user, and the target harsh recorded for that user at the point of registration. If the verification process is successful, then a Boolean value of `True` is returned else `False`. 

The `AuthHandler` class encode a dictionary of parameters to produce a single string. The `AuthHandler` `encode_token` method performs this operation by using the `python-jose` package.  This package can be installed with support for cryptography using the pip command. 

~~~bash
$ pip install python-jose[cryptography]
~~~

The OAuth2 specification recommends some keys to use for encoding but this is not mandatory. In our design we adopt the `sub`, `iat`, and `exp` keys. These recommended keys should be used to encourage interoperability. We also introduced the `cnonce` key which is a randomly generated number or string from the client. Where applicable, the `cnonce` can be shared between the server and client to verify how fresh or recent a token is.

🔍

> The official specification of JWT defines a list of recommended keys which may not be mandatory. Developers can add their own keys but should not use reserved names in the JWT definitions.
> The following JWT keys are recommended but not mandatory.

| Key | Name | Description |
| ----| ---- | -------------|
| iss |Issuer|The name of the service or application issuing the JWT|
| sub | Subject| The subject for which the JWT was generated. This is often the username where the password flow of OAuth2 is being used.|
| aud | Audience| The recipients of the JWT|
| exp | Expiration time | The time limit for the JWT. After this time, the JWT is invalid|
| iat | Issued at time| The time at which the generated JWT was issued|
| nbf | Not before | Indicates the time for which the JWT must not be accepted for processing|
| jti | JWT ID| A unique identifier for the JWT|

> A more detailed list of all possible keys can be found in the IANA [website](https://www.iana.org/assignments/jwt/jwt.xhtml#claims).

Our `AuthHandler` class can decode JWT token for two primary use cases. The first use case is to provide an identifier for a user upon successfully authenticating the user. This is performed by the `AuthHandler` `auth_wrapper` method. This method when called will decode a given token string in the HTTP request Authorization Bearer header. If the operation is successful, the target user is authenticated, and the unique identifier is returned. The second use case is like the first as the HTTP request Authorization Bearer header is used to obtain the token. However, in the second use case, a dictionary object of the credentials used for encoding the JWT token is returned. This case is used where more information like the `scope` and `cnonce` are needed.

A scope is simply a string or list of names (may have URL encoded spaces) used in the JWT token to identify a collection of permissions a user or client can have for the current token. For example, we can define a default scope, named `default`, which allows unregistered users to only access the login, registration, signup, and welcome API endpoints. All other endpoints will not be accessible to this user as they are outside the default user scope. OAuth2 specifies a scope can be provided in authentication form alongside the username and password using the password flow. 

In our designed prototype, the authentication server is embedded in the same application server. In some other setup, authentication may be done by a third-party, example Google. In this scenario, a client public ID (`client_id`) and a client secret (`client_secret`) will be provided for the application. The client ID and secret are only available after duly registering the application with the authentication service. Authenticating a user will then require sending needed parameters like username or email to the specified authentication API endpoint. A sample of this is shown for using Google and Facebook authentication service for a user. This operation entails getting a response that confirms the email address of the user and if valid and the email is associated with a unique user in our application record, we can then generate the needed access token for the user. However, if the email is verified by Google or Facebook but does not exist in our application record, we can proceed to register the user with limited access by generating a token with the default scope. This is shown in the `login_via_handle` function. By calling the `create_user` function using the verified email we create a new user instance with a set of default permissions. We can select between the Google or Facebook authentication service by a flag of integer value. The `get_user_by_identity` takes a verified email and searches for a record of a user that is associated with this email. If a user is found, the function returns the instance of the user, else it returns `None`.

~~~python


def login_via_handle(data: Dict = Body(...), flag: int) -> str:
    token = data.get('accessToken')
    response = None
    if flag == 2:
        response = requests.get(settings.FACEBOOK_LOGIN_URL,
                                params={'fields': 'id,name,email', 'access_token': token}).json()
    elif flag == 3:
        response = requests.get(settings.GOOGLE_LOGIN_URL, params={'access_token': token}).json()

    if email := response.get('email'):
        if user := get_user_by_identity(email):
            return {'access_token': auth.encode_token(user.slug), 'token_type': 'bearer'}
        else:
            user = create_user({'email': response.get('email')})
            return {'access_token': auth.encode_token(user.slug), 'token_type': 'bearer'}
    else:
        raise AppRequestException(status=status.HTTP_404_NOT_FOUND, errors=[error_messages.handle_signin],
                                  success=False)


~~~

If no valid response is received from the authentication service, then we can raise an `ApplicationRequestException` with a corresponding message stating the user cannot be registered or authenticated using the provided email or social handle. 

#### 📚 Further Read - Authentication
Several methods exist for authenticating a user. For example, the OpenID connect builds on the OAuth2 specification and can also be implemented in FastAPI. FastAPI framework provides the security package that enables developers adopt different implementations on authenticating and managing access to resources. Further information on OAuth, OpenID, JWT are listed below.

📖 [OAuth2 Specification](https://datatracker.ietf.org/doc/html/rfc6749)
📖 [How OpenID Connect works](https://openid.net/developers/how-connect-works/)
📖 [RFC 7519 JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)

## Access Control

Authentication and Access Control as shown in OWASP are major security factors to consider in any online or standalone system. Web applications like the mWallet has a greater need for security considering the level of risks involved in managing financial data of multiple users that may be across a wide region. The FastAPI provides tools for developers to implement access control by using standard OAuth2PasswordBearer and similar security classes and to integrate third party plugins. Developers can also design security classes of their own to suit their business needs. In our work so far, we have implemented authentication as part of our application and shown how to authenticate users via Google and Facebook service. The OAuth2 specification defines access control based on a token (an encoded string which may have information contained as key-value pair) and requesting of permission using scopes. We will explore a simplified use of scopes in JWT to define access for a user.

### Permission based on scopes

In the `AuthHandler` encoding operation, a set of parameters are encoded into the JWT. One of these parameters is the scope information. We had earlier started that a scope is simply a list of strings that identifies what permission a given token is assigned. Let us take a deeper look into the `encode_token` method. The `encode_token` method takes a `user` parameter for uniquely identifying users; a `scopes` parameter which is a list of strings; an optional `cnonce` parameter that can be an integer or string value. We define a `payload` dictionary which is a mapping of data to be encoded into the JWT. The `payload` comprises the start and end time. These values are assigned to the `iat` and `exp` keys of the JWT. We then can add the allowed permissions for the JWT using the provided scopes. The list is validated against a database entry to ensure that the given user is in a `Group` that has a matching set of scopes and arbitrary strings that are not defined as scopes will be rejected. This search is performed using our SQLAlchemy `session` object. If our query is successful, then the set of provided scopes are added to the JWT otherwise, we add an empty list as our scope key (that is, our default scope) in the JWT. 

~~~python

import base64
import logging
import re
from datetime import timedelta
from typing import Any, Dict, List, Tuple, TypeVar, Generator

import stringcase
from fastapi import HTTPException, status, Body, Request, Security, Query, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials, SecurityScopes
from jose import jwt
from passlib.context import CryptContext
from sqlalchemy import create_engine, or_, insert
from sqlalchemy.orm import Session

from app_cryptography import key_hashing, verify_key_hashing
from bitmast_mwallet import settings
from db import get_session
from messages import error_messages
from models.administration import User, Group, user_group_association, UserType
from schemas.administration import LoginData
from signals.authentication import generate_otp_signal, sign_in_signal, sign_up_signal, password_reset_signal
from signals.event_bus import EventBus
from util import current_timezone, current_timestamp, thread_runner

engine = create_engine(settings.DB_CONNECTION_URL, echo=True)

logger = logging.getLogger(__name__)

credentials_exception = HTTPException(
    status_code=status.HTTP_401_UNAUTHORIZED,
    detail="Could not validate credentials",
    headers={"WWW-Authenticate": "Bearer"},
)

ALGORITHM = settings.AUTHENTICATION_ALGORITHM
ACCESS_TOKEN_EXPIRE_MINUTES = settings.ACCESS_TOKEN_EXPIRE_MINUTES

class AuthHandler:

    # Excluded other class details

    def encode_token(self, user, scopes: List = None, nonce: int | str = None) -> str:
        start = current_timezone()
        end = start + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
        payload = {
            'exp': end,
            'iat': start,
            'sub': user,
        }

        if isinstance(nonce, (int, float, str)):
            nonce = str(nonce)
            payload.update(cnonce=nonce)

        if scopes:
            s = get_session()
            with s.begin():
                if s.query(Group.scope).join(User).filter(Group.scope.in_(scopes), User.slug == user).all():
                    payload.update(scopes=scopes)
                else:
                    payload.update(scopes=[])
                s.commit()

        return jwt.encode(payload, self.secret, algorithm=settings.AUTHENTICATION_ALGORITHM)   

~~~

Our generated JWT now has information about scope. We may define a scope to support single or multiple permissions. For example, Instagram web API allows clients to request a combination of scopes like `user_profile`, and `user_media` in a single request which grants them access to multiple resources. Another approach could use a single scope string to allow users access multiple resources. Example, a `user_resource` scope may allow a user to view a profile, media, and other resources associated with a user. In the mWallet application, permission to users is granted by assigning the user to a given Group. Each Group has a corresponding list of scopes that can be assigned to them. For instance, an admin user may have access to add entry to a Catalogue of assets but not delete any asset from a Catalogue. A super admin user on the other hand may add, edit, or delete entry to a Catalogue. Also, let's assume an admin user can add a new user but only the registered user can update his or her profile. In effect, our permission system should be able to enable users perform actions based on their Group and the type of Resource in view. To achieve our goal while keeping things simple, we decided to define a permission.yml file and declaratively state what resource can be accessed by a given scope.

~~~yml

# Define the various permission rules for defined user groups and types.
# Permissions are set on a per model basis using named scopes. This allows for different access rights to models for
# a given scope.

permissions:
  - model:
      name: Catalogue
      description: |
        The catalogue permission describes a set of permission for the catalogue model. 
        The permission will allow various users add, edit, view, or delete
        a catalogue. Only user groups assigned permission can perform such task on a catalogue.
        
      permissions:
        - scope:
            name: &super super admin
            action: &super_permission [ delete, edit, view, add, list ]
        - scope:
            name: &admin admin
            action: *super_permission        
        - scope:
            name: &auditor insurer officer
            action: &view_permission [ view ]
        - scope:
            name: &chief_auditor chief auditor
            action: *view_permission        
  - model:
      name: Services
      description: |
        Defines a range of services which are available to various user types. 
        This permission set covers named API endpoints in the action attribute.
      permissions:
        - scope:
            name: *super
            action: &super_admin_services
              [
                # Define services for catalogue approval
                approve_catalogue_change_request,
                revise_catalogue,
                # Define services for catalogue resource
                basic_search,
                # Define services for order approval
                approve_order_auth_code,                
                approve_payment,
                approve_payment_from_account,
                generate_order_authentication_code, 
                view_open_orders,
                view_open_transactions,
               
              ]
        - scope:
            name: *admin
            action: *super_admin_services        
        - scope:
            name: *chief_auditor
            action: [
                # Define services for catalogue approval
                approve_catalogue_change_request,
                revise_catalogue,
                # Define services for catalogue resource
                basic_search,
                # Define services for order approval
                approve_order_auth_code,                
                approve_payment,
                approve_payment_from_account,
                generate_order_authentication_code, 
                view_open_orders,
                view_open_transactions,
            ]
        - scope:
            name: *auditor
            action: [              
              view_open_orders,
              view_open_transactions,
            ]
         
        - scope:
            name: base user
            action: [register, home]

~~~

In our permission file, a `scope` is a mapping having a `name` and `action` key. The `name` key is a string that identifies the scope with a human readable name. The `action` key is a list of possible actions a user can perform if given the scope in view. In effect, we declared a scope in the permission.yml file as having a list of actions. The permission list in our `permission.yml` file holds all possible scope definitions for a given model. The model’s name and description are captured by the corresponding model keys name and description.

Aside defining models and the scope associated with them, we can define API endpoints which may not be linked to any given model. These API endpoints are tagged `Services` in our `permission.yml` file. A Service object can have a list of permissions and corresponding scope definitions just like our model mapping. However, instead of having a list of CRUD operations (`add`, `edit`, `view`, `delete`), the function name for the path operations is provided in the action attribute. This approach enables us to define what API endpoint (that may or may not be associated with a given Resource) a given user group can access.

We now have in place a permission file that informs us of the various actions a given user Group may perform. However, we will also need to parse the `permission.yml` file into data that can be used by our application dynamically. A `Permission` class is defined to consume the `permission.yml` file and generate a registry of scopes that defines actions the user can perform. By centralizing our scopes and dynamically loading them at run time, we can manage user access with minimal changes to our code and reduced read/write to our database. We could also save this information in a database however, since our permission settings are not likely to be changed frequently and should not be changed during an active user session, the file approach is more in line with our use case. In effect, once our FastAPI application is started and all permission settings loaded, user permission or access can be managed programmatically without a possible disruption if the database is being updated or temporarily unavailable. This could be very helpful for endpoints that that does not query the database.

~~~python

import os
from typing import TypeVar, Any, List

import stringcase
import yaml

from db import Base
from models import UserType
from util import extract_data

ModelType = TypeVar('ModelType', bound=Base)

base = os.getcwd()
permissions = os.path.join(base, 'permissions', 'permissions.yml')

messages = None
error_messages = None

class Permission:
    __slots__ = '_permissions', '_scopes', '_registry', '_meta'

    def __init__(self):
        # key in registry is model name or id
        # key points to tuple or dict
        self._scopes = [stringcase.lowercase(x.name) for x in UserType]
        self._meta = {}
        self._registry = {}
        self._permissions = set()

        with open(permissions, 'r') as fh:
            try:
                data = yaml.safe_load(fh)
                actions = []
                for user_permission in data.get('permissions'):
                    model_ = user_permission.get('model')
                    model_name = model_.get('name')
                    model_permission_description = model_.get('description')
                    model_permissions = model_.get('permissions')
                    record = []

                    for permission in model_permissions:

                        permission_action = extract_data('action', permission)
                        permission_scope = extract_data('name', permission)

                        scope = stringcase.snakecase(stringcase.lowercase(permission_scope))

                        record.append((scope, permission_action,))
                        if permission_action:
                            actions += permission_action

                    self._meta.update({model_name: model_permission_description})
                    self._registry.update({model_name: record})

                if actions:
                    permission_set = set()

                    for a in actions:
                        if isinstance(a, str):
                            acts = a.split(' ', -1)
                            permission_set.update(acts)
                        elif isinstance(a, list | tuple):
                            permission_set.update(a)
                    self._permissions = permission_set

            except yaml.YAMLError:
                raise

    def list_permissions(self, model):
        # Extract given field from possibly nested data structure
        data_ = self._registry.copy()
        model_ = model if isinstance(model, str) else model.__name__ if isinstance(model, Base) else None

        permission = data_.get(model_)
        return permission

    def has_permission(self, model: Any, endpoint: str) -> bool:
        permission = self.list_permissions(model)
        return any([x for x, y in permission if endpoint == x or endpoint in y])

    def get_permission(self, model: Any, endpoint: str) -> List:
        permission = self.list_permissions(model)
        if permission:
            return [x for x, y in permission if y and endpoint in y]

    def crud_permissions(self, model: Any) -> List:
        crud = ['delete', 'edit', 'view', 'add', 'list']
        permission = self.list_permissions(model)
        if permission:
            return [x for x, y in permission if y and set(crud) & set(y)]

    @property
    def permissions(self):
        return self._permissions

    @property
    def meta(self):
        return self._meta

~~~

The `Permission` class is initialized without any parameter. Internally, the `__init__` method of this class will load the data read from the `permission.yml` file and setup a registry of permissions using the `_registry` attribute. The `_meta`, `_permissions` and `_scopes` attribute are all initialized to a default value with the `_scopes` being assigned a list of values extracted from the `UserType` enum. If the `_registry` is successfully setup, the class methods `list_permissions`, `has_permission`, `crud_permissions` can then be used to: 
+ List all permissions assigned to a given user group that can be encoded in the scope attribute of our JWT
+ Validate what permission or action can be performed on a resource
+ Validate if CRUD operations are supported by a given model. 

The `crud_permissions` indicates that various user groups can perform an arbitrary set of operation on a resource. For instance, a user group may add a new record to a resource table but cannot delete any record from the resource table. This fine-grained management of resources is suitable where certain resource should not be modified as changes may affect services of other users in the same or different group.

One use case of the `Permission` class is to dynamically determine what scope or set of scopes should be available at each API endpoint when registering routes. From our `permission.yml` file, we can see that the super admin user group can approve request to change a catalogue, revise a change to a catalogue, perform basic search of catalogues, and other services. The default user on the other hand can only access the registration endpoint and view the application home endpoint. We can configure this information and load it into our route definitions, helping us define access control at runtime. Furthermore, changes to our `permission.yml` file will be automatically reloaded into the application routes upon system restart. This approach allows us to centralize management of permissions without affecting existing database records.

## API Permission class
**Using dependencies to enforce permission**

Having the scopes needed per route is one part of the security equation. We also need to consider how to manage access to models to perform CRUD operations that updates our underlying database. Web frameworks like Django, Laravel, or CakePHP provides means by which CRUD operations can be performed from a model class. FastAPI does not come in built with an ORM and as such developers can use an ORM of choice and access any utility available to them towards reaching their use case. We will define a `ManageResource` class that will be used to perform any CRUD operation on a given SQLAlchemy model class. 

~~~python
from enum import Enum
from typing import Dict, Any, Optional, Type, List, Union, Sequence, Callable, Tuple

from fastapi import Body, Query, Path, params, HTTPException
from fastapi.datastructures import DefaultPlaceholder, Default
from fastapi.encoders import SetIntStr, DictIntStrAny
from fastapi.routing import APIRoute
from fastapi.utils import generate_unique_id
from starlette import status
from starlette.responses import Response, JSONResponse
from starlette.routing import BaseRoute

from bhis_claims import settings
from db.adapters import crud_operation, Operation
from util import thread_runner

credential_error = HTTPException(
    status_code=status.HTTP_401_UNAUTHORIZED,
    detail="Unable to process resource for the selected user",
    headers={"WWW-Authenticate": "Bearer"},
)

class ManageResource:
    __slots__ = ('_resource', '_router', '_schema')
    
    class Config:
        CREATE_FLAG = False
        RETRIEVE_FLAG = False
        LIST_FLAG = False
        UPDATE_FLAG = False
        DESTROY_FLAG = False

    def __init__(self, resource, router, schema: Any = None):
        self._resource = resource
        self._router = router
        self._schema = schema

        self.Config.CREATE_FLAG = False
        self.Config.RETRIEVE_FLAG = False
        self.Config.LIST_FLAG = False
        self.Config.UPDATE_FLAG = False
        self.Config.DESTROY_FLAG = False

    def _create_resource(self, data: Any = Body(...)) -> Any:
        result = crud_operation(model_class=self._resource, data=data, operation=Operation.CREATE)
        if isinstance(result, tuple) and len(result) == 1:
            return result[0]

        return result

    @thread_runner(workers=settings.DEFAULT_THREAD_WORKERS)
    def _retrieve_resource(self, identifier: Any = None) -> Any:
        return crud_operation(model_class=self._resource, operation=Operation.READ, identifier=identifier)

    def _list_resource(self, skip: int = Query(None), limit: int = Query(None)) -> List | Tuple:
        return crud_operation(model_class=self._resource, operation=Operation.LIST, skip=skip, limit=limit)

    def _update_resource(self, identifier: Any = Path(None), data: Dict = Body(...), criteria: Dict = Body(None)) -> \
            Tuple[Any, int]:
        result = crud_operation(model_class=self._resource, operation=Operation.UPDATE, identifier=identifier,
                                data=data, filter_by=criteria)
        if isinstance(result, tuple) and len(result) == 1:
            return result[0]

        return result

    def _destroy_resource(self, identifier: Any = Path(None)) -> int:
        result = crud_operation(model_class=self._resource, operation=Operation.DELETE, identifier=identifier)
        if isinstance(result, tuple) and len(result) == 1:
            return result[0]

        return result

    def create_route(self, path, *, response_model: Optional[Type[Any]] = None, status_code: Optional[int] = None,
                     tags: Optional[List[Union[str, Enum]]] = None,
                     dependencies: Optional[Sequence[params.Depends]] = None, summary: Optional[str] = None,
                     description: Optional[str] = None, response_description: str = "Successful Response",
                     responses: Optional[Dict[Union[int, str], Dict[str, Any]]] = None,
                     deprecated: Optional[bool] = None, operation_id: Optional[str] = None,
                     response_model_include: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                     response_model_exclude: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                     response_model_by_alias: bool = True, response_model_exclude_unset: bool = False,
                     response_model_exclude_defaults: bool = False, response_model_exclude_none: bool = False,
                     include_in_schema: bool = True,
                     response_class: Union[Type[Response], DefaultPlaceholder] = Default(JSONResponse),
                     name: Optional[str] = None,
                     callbacks: Optional[List[BaseRoute]] = None, openapi_extra: Optional[Dict[str, Any]] = None,
                     generate_unique_id_function: Union[Callable[[APIRoute], str], DefaultPlaceholder] = Default(
                         generate_unique_id)):

        if self.Config.CREATE_FLAG:
            if not name:
                name = f'{self._resource.__tablename__}_add'
            route = APIRoute(
                path=path,
                endpoint=self._create_resource,
                methods=['POST'],
                name=name,
                response_model=response_model,
                status_code=status_code,
                tags=tags,
                dependencies=dependencies,
                summary=summary,
                description=description,
                response_description=response_description,
                responses=responses,
                deprecated=deprecated,
                operation_id=operation_id,
                response_model_include=response_model_include,
                response_model_exclude=response_model_exclude,
                response_model_by_alias=response_model_by_alias,
                response_model_exclude_unset=response_model_exclude_unset,
                response_model_exclude_defaults=response_model_exclude_defaults,
                response_model_exclude_none=response_model_exclude_none,
                include_in_schema=include_in_schema,
                response_class=response_class,
                callbacks=callbacks,
                openapi_extra=openapi_extra,
                generate_unique_id_function=generate_unique_id_function
            )
            self._router.routes.append(route)
            return route

        else:
            raise credential_error

    def retrieve_route(self, path, *, response_model: Optional[Type[Any]] = None, status_code: Optional[int] = None,
                       tags: Optional[List[Union[str, Enum]]] = None,
                       dependencies: Optional[Sequence[params.Depends]] = None, summary: Optional[str] = None,
                       description: Optional[str] = None, response_description: str = "Successful Response",
                       responses: Optional[Dict[Union[int, str], Dict[str, Any]]] = None,
                       deprecated: Optional[bool] = None, operation_id: Optional[str] = None,
                       response_model_include: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                       response_model_exclude: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                       response_model_by_alias: bool = True, response_model_exclude_unset: bool = False,
                       response_model_exclude_defaults: bool = False, response_model_exclude_none: bool = False,
                       include_in_schema: bool = True,
                       response_class: Union[Type[Response], DefaultPlaceholder] = Default(JSONResponse),
                       name: Optional[str] = None,
                       callbacks: Optional[List[BaseRoute]] = None, openapi_extra: Optional[Dict[str, Any]] = None,
                       generate_unique_id_function: Union[Callable[[APIRoute], str], DefaultPlaceholder] = Default(
                           generate_unique_id)):
        if self.Config.RETRIEVE_FLAG:
            if not name:
                name = f'{self._resource.__tablename__}_view'

            route = APIRoute(
                path=path,
                endpoint=self._retrieve_resource,
                methods=['GET'],
                name=name,
                response_model=response_model,
                status_code=status_code,
                tags=tags,
                dependencies=dependencies,
                summary=summary,
                description=description,
                response_description=response_description,
                responses=responses,
                deprecated=deprecated,
                operation_id=operation_id,
                response_model_include=response_model_include,
                response_model_exclude=response_model_exclude,
                response_model_by_alias=response_model_by_alias,
                response_model_exclude_unset=response_model_exclude_unset,
                response_model_exclude_defaults=response_model_exclude_defaults,
                response_model_exclude_none=response_model_exclude_none,
                include_in_schema=include_in_schema,
                response_class=response_class,
                callbacks=callbacks,
                openapi_extra=openapi_extra,
                generate_unique_id_function=generate_unique_id_function
            )
            self._router.routes.append(route)
            return route
        else:
            raise credential_error

    def list_route(self, path, *, response_model: Optional[Type[Any]] = None, status_code: Optional[int] = None,
                   tags: Optional[List[Union[str, Enum]]] = None,
                   dependencies: Optional[Sequence[params.Depends]] = None, summary: Optional[str] = None,
                   description: Optional[str] = None, response_description: str = "Successful Response",
                   responses: Optional[Dict[Union[int, str], Dict[str, Any]]] = None,
                   deprecated: Optional[bool] = None, operation_id: Optional[str] = None,
                   response_model_include: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                   response_model_exclude: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                   response_model_by_alias: bool = True, response_model_exclude_unset: bool = False,
                   response_model_exclude_defaults: bool = False, response_model_exclude_none: bool = False,
                   include_in_schema: bool = True,
                   response_class: Union[Type[Response], DefaultPlaceholder] = Default(JSONResponse),
                   name: Optional[str] = None,
                   callbacks: Optional[List[BaseRoute]] = None, openapi_extra: Optional[Dict[str, Any]] = None,
                   generate_unique_id_function: Union[Callable[[APIRoute], str], DefaultPlaceholder] = Default(
                       generate_unique_id)):

        if self.Config.LIST_FLAG:
            if not name:
                name = f'{self._resource.__tablename__}_list'

            route = APIRoute(
                path=path,
                endpoint=self._list_resource,
                methods=['GET'],
                name=name,
                response_model=response_model,
                status_code=status_code,
                tags=tags,
                dependencies=dependencies,
                summary=summary,
                description=description,
                response_description=response_description,
                responses=responses,
                deprecated=deprecated,
                operation_id=operation_id,
                response_model_include=response_model_include,
                response_model_exclude=response_model_exclude,
                response_model_by_alias=response_model_by_alias,
                response_model_exclude_unset=response_model_exclude_unset,
                response_model_exclude_defaults=response_model_exclude_defaults,
                response_model_exclude_none=response_model_exclude_none,
                include_in_schema=include_in_schema,
                response_class=response_class,
                callbacks=callbacks,
                openapi_extra=openapi_extra,
                generate_unique_id_function=generate_unique_id_function
            )
            self._router.routes.append(route)
            return route
        else:
            raise credential_error

    def destroy_route(self, path, *, response_model: Optional[Type[Any]] = None, status_code: Optional[int] = None,
                      tags: Optional[List[Union[str, Enum]]] = None,
                      dependencies: Optional[Sequence[params.Depends]] = None, summary: Optional[str] = None,
                      description: Optional[str] = None, response_description: str = "Successful Response",
                      responses: Optional[Dict[Union[int, str], Dict[str, Any]]] = None,
                      deprecated: Optional[bool] = None, operation_id: Optional[str] = None,
                      response_model_include: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                      response_model_exclude: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                      response_model_by_alias: bool = True, response_model_exclude_unset: bool = False,
                      response_model_exclude_defaults: bool = False, response_model_exclude_none: bool = False,
                      include_in_schema: bool = True,
                      response_class: Union[Type[Response], DefaultPlaceholder] = Default(JSONResponse),
                      name: Optional[str] = None,
                      callbacks: Optional[List[BaseRoute]] = None, openapi_extra: Optional[Dict[str, Any]] = None,
                      generate_unique_id_function: Union[Callable[[APIRoute], str], DefaultPlaceholder] = Default(
                          generate_unique_id)):
        if self.Config.DESTROY_FLAG:
            if not name:
                name = f'{self._resource.__tablename__}_delete'

            route = APIRoute(
                path=path,
                endpoint=self._destroy_resource,
                methods=['DELETE'],
                name=name,
                response_model=response_model,
                status_code=status_code,
                tags=tags,
                dependencies=dependencies,
                summary=summary,
                description=description,
                response_description=response_description,
                responses=responses,
                deprecated=deprecated,
                operation_id=operation_id,
                response_model_include=response_model_include,
                response_model_exclude=response_model_exclude,
                response_model_by_alias=response_model_by_alias,
                response_model_exclude_unset=response_model_exclude_unset,
                response_model_exclude_defaults=response_model_exclude_defaults,
                response_model_exclude_none=response_model_exclude_none,
                include_in_schema=include_in_schema,
                response_class=response_class,
                callbacks=callbacks,
                openapi_extra=openapi_extra,
                generate_unique_id_function=generate_unique_id_function
            )
            self._router.routes.append(route)
            return route
        else:
            raise credential_error

    def update_route(self, path, *, response_model: Optional[Type[Any]] = None, status_code: Optional[int] = None,
                     tags: Optional[List[Union[str, Enum]]] = None,
                     dependencies: Optional[Sequence[params.Depends]] = None, summary: Optional[str] = None,
                     description: Optional[str] = None, response_description: str = "Successful Response",
                     responses: Optional[Dict[Union[int, str], Dict[str, Any]]] = None,
                     deprecated: Optional[bool] = None, operation_id: Optional[str] = None,
                     response_model_include: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                     response_model_exclude: Optional[Union[SetIntStr, DictIntStrAny]] = None,
                     response_model_by_alias: bool = True, response_model_exclude_unset: bool = False,
                     response_model_exclude_defaults: bool = False, response_model_exclude_none: bool = False,
                     include_in_schema: bool = True,
                     response_class: Union[Type[Response], DefaultPlaceholder] = Default(JSONResponse),
                     name: Optional[str] = None,
                     callbacks: Optional[List[BaseRoute]] = None, openapi_extra: Optional[Dict[str, Any]] = None,
                     generate_unique_id_function: Union[Callable[[APIRoute], str], DefaultPlaceholder] = Default(
                         generate_unique_id)):
        if self.Config.UPDATE_FLAG:
            if not name:
                name = f'{self._resource.__tablename__}_edit'

            route = APIRoute(
                path=path,
                endpoint=self._update_resource,
                methods=['PUT'],
                name=name,
                response_model=response_model,
                status_code=status_code,
                tags=tags,
                dependencies=dependencies,
                summary=summary,
                description=description,
                response_description=response_description,
                responses=responses,
                deprecated=deprecated,
                operation_id=operation_id,
                response_model_include=response_model_include,
                response_model_exclude=response_model_exclude,
                response_model_by_alias=response_model_by_alias,
                response_model_exclude_unset=response_model_exclude_unset,
                response_model_exclude_defaults=response_model_exclude_defaults,
                response_model_exclude_none=response_model_exclude_none,
                include_in_schema=include_in_schema,
                response_class=response_class,
                callbacks=callbacks,
                openapi_extra=openapi_extra,
                generate_unique_id_function=generate_unique_id_function
            )
            self._router.routes.append(route)
            return route
        else:
            raise credential_error

    @property
    def router(self):
        return self._router

class CreateMixin(ManageResource):
    class Config:
        CREATE_FLAG = True

class RetrieveMixin(ManageResource):
    class Config:
        RETRIEVE_FLAG = True

class UpdateMixin(ManageResource):
    class Config:
        UPDATE_FLAG = True

class DestroyMixin(ManageResource):
    class Config:
        DESTROY_FLAG = True

class ListMixin(ManageResource):
    class Config:
        LIST_FLAG = True
        RETRIEVE_FLAG = True

~~~

The `ManageResource` defines an inner `Config` class along with a set of attributes `_resource`, `_schema`, and `_router`. The `Config` class is used to set flags for CRUD operations. The named flags `CREATE_FLAG`, `RETRIEVE_FLAG`,       `LIST_FLAG`, `UPDATE_FLAG`, `DESTROY_FLAG` all have a default value of `False`. The `_resource` attribute holds a reference to the given SQLAlchemy ORM model which a CRUD operation is being performed on. The `_schema` attribute holds a reference to the pydantic model class that is associated with the given resource or SQLAlchemy ORM model. This schema class may be dynamically generated as shown earlier using the `make_schema` class. (See Defining and Generating schemas section in [Chapter 5](../chp5/chp5.md) - The business domain entities and database layer).

The `ManageResource` also defines a set of methods used as path operation functions, that is, functions or other callables that will handle HTTP request sent to a given URL. The defined CRUD operation path will rely on these functions for handling user\92s requests to create, read, update, or delete a resource. These methods `_create_resource`, `_retrieve_resource`, `_update_resource`, `_destroy_resource`, and `_list_resource` is used internally by the `ManageResource` class to define FastAPI `APIRoute` instances which can be added to our FastAPI application through the `add_api_route` or `add_route` methods as needed. In our definition, the `ManageResource._create_resource`, `ManageResource._retrieve_resource`, `ManageResource._update_resource`, `ManageResource._destroy_resource`, and `ManageResource._list_resource` are the endpoint functions called by the respective routes to add, view, edit, delete or list all instances of the specified model. This is achieved in the `ManageResource.create_route`, `ManageResource.retrieve_route`, `ManageResource.update_route`, `ManageResource.destroy_route`, and `ManageResource.list_route` which initializes the respective APIRoute instances. By using this approach, we can simply call the `ManageReource` methods to return a route that will be added to the application. For example, the `ManageResource.create_route` will return a FastAPI APIRoute instance which can be added to the FastAPI application. 

Our `ManageResource` class can generate various routes to perform CRUD operations, however, we do not yet have fine grained access control. At the current state, all CRUD operations are available to any group of users. So, what if we want fine grained control of what CRUD operation can be performed on a resource? To achieve this, we can extend the `ManageResource` and put measures in place that will enforce access based on a set of flags received. 

We defined the `APIRouteResource` class to extend `ManageResource`. Operations for CRUD services will be managed by the base class but the derived class, `APIRouteResource`, will manage what CRUD operation will be available and dynamically add permissions defined in our `permission.yml` file through a `Permission` object. This will enable us to achieve a mixins like behaviours where we can select what CRUD operations are available based on the specified resource. For example, we can have scenarios where only read operations (retrieve or list resources) are available and all write operations (update or delete resources) are not available. Furthermore, we can customise the behaviour of the given method by using the `Permission` object to restrict access control based on `Group` assigned to the current user. We will look at various segments of the `APIRouteResource` and the `PermissionResource` to see how this is achieved. 

~~~python
from enum import IntFlag
from typing import TypeVar, Any, get_type_hints, Callable, TYPE_CHECKING, NoReturn, List, MutableSequence, Tuple

from fastapi import Security, HTTPException
from fastapi.routing import APIRoute
from fastapi_restful.inferring_router import InferringRouter
from starlette import status

from authentication.services.jwt_handler import get_user_access
from db import Base
from db.resource import ManageResource
from db.schemas import make_schema
from permissions import Permission
from routers import app_routes
from schemas.base import DefaultResponse

ModelT = TypeVar('ModelT', bound=Base)

class ResourceFlags(IntFlag):
    CREATE = 2
    RETRIEVE = 3
    UPDATE = 5
    DESTROY = 7
    LIST = 11

credential_error = HTTPException(
    status_code=status.HTTP_401_UNAUTHORIZED,
    detail="Unable to process resource for the selected user",
    headers={"WWW-Authenticate": "Bearer"},
)

class APIRouteResource(ManageResource):
    app_permissions = Permission()

    def __init__(self, resource, router, schema: Any = None, flags: int | ResourceFlags | List = 0):

        super().__init__(resource, router, schema)

        match flags:
            case ResourceFlags.CREATE | 2:
                self.Config.CREATE_FLAG = True
            case ResourceFlags.UPDATE | 5:
                self.Config.UPDATE_FLAG = True
            case ResourceFlags.DESTROY | 7:
                self.Config.DESTROY_FLAG = True
            case ResourceFlags.RETRIEVE | 3:
                self.Config.RETRIEVE_FLAG = True
            case ResourceFlags.LIST | 11:
                self.Config.LIST_FLAG = True
            case [ResourceFlags.RETRIEVE, ResourceFlags.LIST]:
                self.Config.RETRIEVE_FLAG = True
                self.Config.LIST_FLAG = True
            case [ResourceFlags.CREATE, ResourceFlags.UPDATE, ResourceFlags.DESTROY]:
                self.Config.CREATE_FLAG = True
                self.Config.DESTROY_FLAG = True
                self.Config.UPDATE_FLAG = True
            case [ResourceFlags.CREATE, ResourceFlags.RETRIEVE, ResourceFlags.UPDATE, ResourceFlags.DESTROY,
                  ResourceFlags.LIST]:
                self.Config.CREATE_FLAG = True
                self.Config.RETRIEVE_FLAG = True
                self.Config.DESTROY_FLAG = True
                self.Config.LIST_FLAG = True
                self.Config.UPDATE_FLAG = True
            case _ as fls:
                # evaluate any possible combination of flags.
                # if flag name has create, then call create_route etc.
                if isinstance(fls, MutableSequence):
                    if ResourceFlags.CREATE in fls:
                        self.Config.CREATE_FLAG = True
                    if ResourceFlags.UPDATE in fls:
                        self.Config.UPDATE_FLAG = True
                    if ResourceFlags.LIST in fls:
                        self.Config.LIST_FLAG = True
                    if ResourceFlags.DESTROY in fls:
                        self.Config.DESTROY_FLAG = True
                    if ResourceFlags.RETRIEVE in fls:
                        self.Config.RETRIEVE_FLAG = True

    @classmethod
    def api_readonly(cls, resource, router, schema, **kwargs):
        instance = cls(resource, router, schema)
        instance.Config.LIST_FLAG = True
        instance.Config.RETRIEVE_FLAG = True
        all((instance.list_route(**kwargs), instance.retrieve_route(**kwargs)))
        return instance

    @classmethod
    def api_write(cls, resource, router, schema, **kwargs):
        instance = cls(resource, router, schema)
        instance.Config.CREATE_FLAG = True
        instance.Config.UPDATE_FLAG = True
        instance.Config.DESTROY_FLAG = True
        all((instance.create_route(**kwargs), instance.update_route(**kwargs), instance.destroy_route(**kwargs)))
        return instance

    @classmethod
    def api_read_write(cls, resource, router, schema, **kwargs):
        instance = cls(resource, router, schema)
        instance.Config.CREATE_FLAG = True
        instance.Config.RETRIEVE_FLAG = True
        instance.Config.DESTROY_FLAG = True
        instance.Config.LIST_FLAG = True
        instance.Config.UPDATE_FLAG = True
        all((instance.create_route(**kwargs), instance.retrieve_route(**kwargs), instance.destroy_route(**kwargs),
             instance.list_route(**kwargs), instance.update_route(**kwargs)))
        return instance

    def _process_route(self, flag):
        search = None
        scopes = None
        if flag == ResourceFlags.CREATE:
            scopes = self.app_permissions.get_permission(self._resource.__name__, endpoint='add')
            search = (x for x in app_routes.routes if x.resource == self._resource.__name__ and x.crud_type == 'add')
        elif flag == ResourceFlags.LIST:
            scopes = self.app_permissions.get_permission(self._resource.__name__, endpoint='list')
            search = (x for x in app_routes.routes if x.resource == self._resource.__name__ and x.crud_type == 'list')
        elif flag == ResourceFlags.UPDATE:
            scopes = self.app_permissions.get_permission(self._resource.__name__, endpoint='edit')
            search = (x for x in app_routes.routes if x.resource == self._resource.__name__ and x.crud_type == 'edit')
        elif flag == ResourceFlags.DESTROY:
            scopes = self.app_permissions.get_permission(self._resource.__name__, endpoint='delete')
            search = (x for x in app_routes.routes if x.resource == self._resource.__name__ and x.crud_type == 'delete')
        elif flag == ResourceFlags.RETRIEVE:
            scopes = self.app_permissions.get_permission(self._resource.__name__, endpoint='view')
            search = (x for x in app_routes.routes if x.resource == self._resource.__name__ and x.crud_type == 'view')

        return scopes, search

    def create_route(self, **kwargs):
        scopes, search = self._process_route(flag=ResourceFlags.CREATE)
        try:
            # create route and add to router
            route = next(search)
            permission_create_route = APIRoute(
                path=route.url_path,
                methods=route.methods,
                endpoint=self._create_resource,
                tags=route.tags,
                description=route.description,
                name=route.name,
                status_code=route.status,
                response_model=make_schema(self._resource),
                response_description=route.success,
                dependencies=[Security(get_user_access, scopes=scopes)],
                **kwargs
            )
            # add to router
            self._router.routes.append(permission_create_route)
            return permission_create_route
        except StopIteration:
            pass

    def retrieve_route(self, **kwargs):
        scopes, search = self._process_route(flag=ResourceFlags.RETRIEVE)

        try:
            route = next(search)
            permission_retrieve_route = APIRoute(
                path=route.url_path,
                methods=route.methods,
                endpoint=self._retrieve_resource,
                tags=route.tags,
                description=route.description,
                name=route.name,
                status_code=route.status,
                response_model=make_schema(self._resource),
                response_description=route.success,
                dependencies=[Security(get_user_access, scopes=scopes)],
                **kwargs
            )
            # add to router
            self._router.routes.append(permission_retrieve_route)
            return permission_retrieve_route
        except StopIteration:
            pass

    def update_route(self, **kwargs):
        scopes, search = self._process_route(flag=ResourceFlags.UPDATE)

        try:
            route = next(search)
            permission_update_route = APIRoute(
                path=route.url_path,
                methods=route.methods,
                endpoint=self._update_resource,
                tags=route.tags,
                description=route.description,
                name=route.name or f'{self._resource.__tablename__}_edit',
                status_code=route.status,
                response_model=make_schema(self._resource),
                response_description=route.success,
                dependencies=[Security(get_user_access, scopes=scopes)],
                **kwargs
            )
            # add to router
            self._router.routes.append(permission_update_route)
            return permission_update_route
        except StopIteration:
            pass

    def destroy_route(self, **kwargs):
        scopes, search = self._process_route(ResourceFlags.DESTROY)

        try:
            route = next(search)
            permission_destroy_route = APIRoute(
                path=route.url_path,
                methods=route.methods,
                endpoint=self._create_resource,
                tags=route.tags,
                description=route.description,
                name=route.name or f'{self._resource.__tablename__}_delete',
                status_code=route.status,
                response_model=make_schema(self._resource),
                response_description=route.success,
                dependencies=[Security(get_user_access, scopes=scopes)],
                **kwargs
            )
            # add to router
            self._router.routes.append(permission_destroy_route)
            return permission_destroy_route
        except StopIteration:
            pass

    def list_route(self, **kwargs):
        scopes, search = self._process_route(ResourceFlags.LIST)
        try:
            route = next(search)
            permission_list_route = APIRoute(
                path=route.url_path,
                methods=route.methods,
                endpoint=self._create_resource,
                tags=route.tags,
                description=route.description,
                name=route.name or f'{self._resource.__tablename__}_list',
                status_code=route.status,
                response_model=make_schema(self._resource),
                response_description=route.success,
                dependencies=[Security(get_user_access, scopes=scopes)],
                **kwargs
            )
            # add to router
            self._router.routes.append(permission_list_route)
            return permission_list_route
        except StopIteration:
            pass

class PermissionResource:
    # For the various models and their permission, check if the model permission matches the user group
    

    __slots__ = '_router', '_resource'
    app_permissions = Permission()

    def __init__(self, resource: ModelT, router: Any):
        self._router = router
        self._resource = resource

    @classmethod
    def process_crud(cls, resource, router, flags: int | ResourceFlags | List = None, **kwargs):
        if flags == 0:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Unable to process resource for the selected user",
                headers={"WWW-Authenticate": "Bearer"},
            )

        schema = make_schema(resource)
        inner = APIRouteResource(resource=resource, router=router, schema=schema, flags=flags)

        match flags:
            case ResourceFlags.CREATE | 2:
                inner.create_route(**kwargs)
            case ResourceFlags.UPDATE | 5:
                inner.update_route(**kwargs)
            case ResourceFlags.DESTROY | 7:
                inner.destroy_route(**kwargs)
            case ResourceFlags.RETRIEVE | 3:
                inner.retrieve_route(**kwargs)
            case ResourceFlags.LIST | 11:
                inner.list_route(**kwargs)
            case [ResourceFlags.RETRIEVE, ResourceFlags.LIST]:
                APIRouteResource.api_readonly(resource, router, schema, **kwargs)
            case [ResourceFlags.CREATE, ResourceFlags.UPDATE, ResourceFlags.DESTROY]:
                APIRouteResource.api_write(resource, router, schema, **kwargs)
            case [ResourceFlags.CREATE, ResourceFlags.RETRIEVE, ResourceFlags.UPDATE, ResourceFlags.DESTROY,
                  ResourceFlags.LIST]:
                APIRouteResource.api_read_write(resource, router, schema, **kwargs)
            case _ as fls:
                if isinstance(fls, (List, Tuple)):
                    if ResourceFlags.CREATE in fls:
                        inner.create_route(**kwargs)
                    if ResourceFlags.UPDATE in fls:
                        inner.update_route(**kwargs)
                    if ResourceFlags.LIST in fls:
                        inner.list_route(**kwargs)
                    if ResourceFlags.DESTROY in fls:
                        inner.destroy_route(**kwargs)
                    if ResourceFlags.RETRIEVE in fls:
                        inner.retrieve_route(**kwargs)

~~~

Let us study the `APIRouteResource` further. The `__init__` method of the `APIRouteResource` calls `super().__init__` to setup its attributes as defined in the parent class `ManageResource`. This call will ensure the `_resource`, `_schema`, and `_router` attributes are properly setup, and all Config flags are set to `False`. We can then further customise the instance of `APIRouteResource` by using `match` pattern to check what `ResourceFlags` value was specified. Depending on the flag value, the instance of the `APIRouteResource` can set the corresponding `Config` flag to `True`. For example, if the `ResourceFlags.CREATE` was provided as flags, then the `Config.CREATE_FLAG` is set to `True`. Using this approach all `Config` flag attributes can be set following the `match` and `case` block. A default `case` block is set which evaluates if a sequence (list or tuple) of `ResourceFlags` is provided as a flag. In this `case`, any arbitrary set of `Config` flag attributes can be set to `True` depending on values provided in the sequence. 

#### Further Read - Python `super`
📢 "Return a proxy object that delegates method calls to a parent or sibling class of type. This is useful for accessing inherited methods that have been overridden in a class. The object_or_type determines the method resolution order to be searched. The search starts from the class right after the type". - from [Python official documentation](https://docs.python.org/3/library/functions.html#super)
> The `super` builtin function in Python may seem similar to other OOP language construct (example, super keyword in Java) for calling a method of a parent class. However, in Python, the `super` does not simply call a parent class in the inheritance hierachy. The super function can be used to specify an actual parent class to call by specifying the name of the class. The `super()` could be seen more as a function that generates a linear order of parent classes to select what parent is next in the execution order of a method. This linear order is technically called Method Resolution Order and is defined by an object's  `__mro__` method inherited from type.

📖 [Python built-in super](https://docs.python.org/3/library/functions.html#super)
📖 [Python's super() considered super!](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/) by Raymond Hattinger

We also defined class methods as utility methods that can initialize an instance of `APIRouteResource` as read only instance via `api_read_only` method; write only instance via `api_write` method; or read and write instance via `api_read_write` method. With these methods we can define routes which can perform CRUD operations on the specified resource provided as SQLAlchemy models.

To manage access control while using the `APIRouteResource`, the `_process_route` is defined to retrieve a list of scopes assigned to a given resource. Using a `Permission` object, a set of permission for a given type of resource is obtained by calling the `get_permission` method. Also, the allowed CRUD operation as defined in our `permission.yml` file for the resource can be obtained by calling the routes property of the `app_routes` object. This object as shown earlier holds a register of all API endpoints in the application. We obtain all approved CRUD endpoints for the given resource and all permissions for the resource as `search` and `scopes` variable that is returned by the `_process_route` method. The `search` and `scopes` variable returned by `_process_route` can be used to initialize instances of `APIRoute` in the overridden methods `create_route`, `retrieve_route`, `update_route`, `destroy_route`, `list_route`. These methods will initialize an `APIRoute` and add the needed security dependencies to enforce authentication and authorization for all CRUD operations using the `APIRoute.__init__()` `dependecies` keyword parameter.

Now we have the `APIRouteResource` in place. We need a means by which we can call defined methods of the class to setup CRUD operations dynamically. We will use the `PermissionResource` class for this purpose. This class defines the `process_crud` method which like the `APIRouteResouce` uses a pattern matching to select what instance of `APIRouteResource` to initialize and then call the needed CRUD method (`create_route`, `retrieve_route`, `update_route`, `destroy_route`, or `list_route`). With this in place, we can dynamically initialize routes with appropriate permissions and endpoint data which are then added to the FastAPI application instance. This removes the overhead of defining respective functions for various models in our application domain. We can simply pass the class or class name as a string to the ManageResource and let it produce the needed routes for us.

~~~python

from typing import Dict, List, Any

from fastapi import Security, Body, Query

import schemas
from authentication.services.jwt_handler import get_user_access
from catalogue.services import catalogue as catalogue_services
from models import Catalogue, CatalogueLineItem
from permissions import Permission
from routers import app_routes
from routers.api import PermissionResource, InferringSchemaRouter, ResourceFlags


acl = Permission()

service = 'Services'
prefix = 'catalogue'

catalogue_router = InferringSchemaRouter()
catalogue_item_router = InferringSchemaRouter()

r1 = app_routes.find(endpoint='basic_search')

def basic_search(token: str = Security(get_user_access, scopes=acl.get_permission(service, 'basic_search')),
                 data: schemas.CatalogueSearch | Dict = Body(...), skip: int = Query(None), limit: int = Query(None)) -> List:
    if token:
        return catalogue_services.basic_search(data=data, skip=skip, limit=limit)

catalogue_router.add_api_route(r1.url_path, endpoint=basic_search, description=r1.description,
                               status_code=r1.status, methods=r1.methods, response_description=r1.success,
                               response_model=List[Any])

# Set up application CRUD services for Catalogue and CatalogueLineItem
catalogue_resource = PermissionResource.process_crud(resource=Catalogue, router=catalogue_router,
                                                     flags=[ResourceFlags.LIST, ResourceFlags.RETRIEVE])

catalogue_item_resource = PermissionResource.process_crud(resource=CatalogueLineItem, router=catalogue_item_router,
                                                          flags=[x for x in ResourceFlags])

														  
~~~

We can now add a sample resource to our application. We will be adding the Catalogue resource using the `PermissionResource`. In our code sample above, we obtain a `Permission` object as acl and a registry of API endpoints using the `api_routes` object. We also define a `catalogue_router` and `catalogue_item_router`. These router objects will be used to append `Catalogue` related routes to the application. All the routes for `Catalogue` can be retrieved from the `api_routes`. As an example, we retrieved a route for `basic_search` as `r1` using the `find()` method of `api_routes`. To setup our CRUD endpoints, the `PermissionResource.process_crud` class method is called. This call will initialize all needed CRUD operations as specified in `routers.yml` and with matching permissions as specified in `permission.yml`.
