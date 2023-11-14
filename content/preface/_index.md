+++
title = 'Preface'
date = 2023-11-06T21:31:41+01:00
weight = 1
draft = true
+++


This book was produced based on lessons learned while designing products for Bi
tmast Digital Services Ltd, tutorials for
Codeskol and Boot camp training, and apps for A17S clients. This book is intended for developers who have some
experience of Python programming and a background in web application development using frameworks such as Django or
Flask. An effort will be made to teach FastAPI framework fundamentals and demonstrate alternate techniques to completing
selected tasks. Example of numerous API route addition methods. The notion of online application development utilizing
FastAPI will be explained using a Do-It-Yourself method by providing design recommendations for mWallet, a
cryptocurrency wallet management system that will use Blockcypher, PayStack, and a currency trading API.

I started working with FastAPI about 2021, after a friend introduced me to the framework. I took a look at it and was
honestly struck by the simplicity and excellent documentation. However, upon trying to work on projects with my peers, I
realised there were not so many Python web developers in my network and the few who were into Python used Django
extensively. FastAPI was great but my community was just not aware of its simplicity and possibly best use cases. In
some sense, this was understandable as there was a trend for building web applications in my region. The conventional
web application needs user management, some collection of forms, a relational database system to store data and of
course an administrative user dashboard. Such features are included in Django but not readily available for microservice
frameworks like FastAPI. However, there is indeed a very great use case for FastAPI: microservices and lightweight web
applications that can be easily integrated with Python packages like Machine Learning tools. For this, I found it easier
to use FastAPI than Django - my personal opinion. In this book, I wish to teach FastAPI and address some business needs
common in my region. A number of business cases were considered, and polls taken from an IT community using WhatsApp.
From the selection, the topic addressing Payment was selected. To make for a more interesting read, both fiat and
cryptocurrency payment systems were selected. The core idea of this book is not to simply discuss a technology, but
rather to help programmers learn various skills and tools that can help them build faster and more robust applications,
in this case, web applications using FastAPI.

In writing this book, I had in mind an audience of young programmers at beginner or intermediate skill levels of Python
programming. The sample code used in this book will be drawn from my experience in training and working on short term
projects. My experience in training is drawn from running short tutorials (called Codeskol) for people who are often
self-taught or just fiddling with the idea of coding. This was having a mix of people all with a laptop and oftentimes
smart enough to hack away at any problem. My peers were mainly self-taught, managed multiple jobs, usually running a
tight timetable, many of them have been programming using video tutorials, and had access to blogs, quick fixes, and
tutorial tips on how to get certain things done. When teaching someone with a Stack Overflow account, you simply wonder
if they are learning or weighing your knowledge. One thing was however lacking amongst participants in my tutorial:
craftsmanship! Do not get me wrong, these people were really bright and intelligent but with copying-and-pasting codes
from the Internet and not knowing underlying principles or design patterns it was inevitable to see code which worked
but was almost impossible to manage. It was not always a pleasing experience reading code in which different mindsets
were being portrayed in no harmonious manner. Things just simply work either by sheer luck or simply because the right
sort of problem had not been thrown at the code. I knew breaking the wrong programming habit was as vital as teaching
the right programming principle or passing appropriate technical information.

The real challenge in writing this book is not only on addressing how We Work together, We Learn together, and We Play
together; but also, how to pass technical skill in a concise manner quickly such that people can access “quick fixes”
and still become excellent craftsmen. It really seemed a paradox as becoming a great craftsman demanded intensive focus,
enjoying what one does, and gathering experience overtime. Matching that against a tight schedule seemed daunting but
that is really what I had to deal with and the motivation behind the 100 PAGES SERIES. This book is an attempt to pass
technical information in a hurry without people missing out on the key concepts in helping them solve their problems or
building a solution and then moving on to something else. Keeping in the style of Codeskol, the topics discussed here
are tailored towards building a solution for business needs, and tips gained from developing this solution are presented
in here.

I hope the lessons gathered from our programming experience saves you the reader lots of hassles in building your own
solutions. Sadly, in some cases, code bases for the solution are not publicly available for multiple reasons peculiar to
client’s approval and Nigerian Intellectual Property needs. I however have provided some snippets. In the meantime, I
hope this makes a great read. The reader should note that some examples used in this book may not match actual code in
repository. The code in this book were modified to suit a given concept or idea being explained. Furthermore, using
actual code base could introduce topics outside the scope of this book.

## How the book is organised

The focus of this book is to help gain skills in using FastAPI. With that in mind, I have attempted to showcase tips
from experience in building a project using FastAPI. Each Chapter will focus on an aspect of the selected project with
highlights on design considerations, Software Engineering skills, or related information that may come in handy during a
real life project. All code snippets will be based on Python 3.10. The preferred database Object Relational Mapper (ORM)
used in this book is the SQLAlchemy (version 2.0 or later) and sample code will use both the Core and ORM functions of
SQLAlchemy.

The convention used in this book are as follows:

Names of variables or snippets of code are in red and with Courier New font type. Example:

~~~python

from wallet import services


async def generate_wallet(user: str | uuid.UUID) -> Any:


    return services.generate_wallet(user)

~~~

As this book was generated using markdown, links to resources may be formatted using my default rendering engine and not
in the common blue colour with underlined text. See example link below with the word "codeskol" as a link:

visit [codeskol](http://www.codeskol.com/blog) directs you to web page at URL http://www.codeskol.com/blog

Quotes in each chapter will normally be provided at the end section of each chapter. Example:
> FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.7+ based on standard Python
type hints.

#### Further Reading

The further reading section will provide links to articles, videos, or other resources which a reader may explore to
better understand a topic. Further Reading may be inserted at different positions of a page but
more often at the end of the topic. This was done to reduce the number of times a reader may leave a page to view more information.
Sections on further reading are provided as block quotes.

#### Added tips and tricks

Alternative means of doing a programming task will be showcased as tips and tricks. These tips may not necessarily be
shorter nor shown as better. They are primarily added to broaden the perspective of the reader and prompt readers to
explore alternatives in getting their job done. Most tips and tricks presented here were gathered from live code in
building the mWallet application or sourced across various online forums like Stack Overflow.

**Summary**
FastAPI is a Python web framework by Sebastián Ramírez . Latest version of FastAPI at the time of writing this book is
FastAPI release tag/version 0.96.0.

While care has been taken to showcase a reasonable amount of information to help developers across various chapters, the
RESOURCES chapter covers a variety of Software Engineering topics in building web applications (apps) using Python in
general and FastAPI in particular. Business considerations for running ICT business in Nigeria is also added along with
other resources. Overall, I hope information contained in this book provides meaningful tips to any developer in any
environment but at the moment of writing this book, I had the Nigerian ICT environment in mind.


