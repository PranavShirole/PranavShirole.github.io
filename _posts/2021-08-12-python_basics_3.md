---
layout: post
title: Python Basics - Part 3
tags: [python]
---

This is Part 3 of a Python tutorial for beginners.

This blog post is a continuation of the previous post *Python Basics - Part 2*.

---
### More on Functions

We know that functions can have inputs, some functionality, and outputs. One thing to note is that functions are first-class objects, i.e. they can be passed around as arguments, just like `int`, `string`, `float`, etc. That means that you can take existing functions and build other functions that use the existing functions.


```python
# these are our existing functions

def add(n1, n2):
    return n1 + n2

def subtract(n1, n2):
    return n1 - n2

def multiply(n1, n2):
    return n1 * n2

def divide(n1, n2):
    return n1 / n2
```


```python
# this is a new function

def calculate(calc_function, n1, n2):
    return calc_function(n1, n2)
    
result = calculate(add, 5, 8)
print(result)
```

    13
    


```python
result2 = calculate(multiply, 5, 8)
print(result2)
```

    40
    

#### Nested Functions

Functions can be nested inside other functions.


```python
def outer_function():
    print("I'm outer")
    
    def nested_function():
        print("I'm inner")
    
    nested_function()  # function being activated here

outer_function()
```

    I'm outer
    I'm inner
    

Functions can also be returned from other functions.


```python
def outer_function():
    print("I'm outer")
    
    def nested_function():
        print("I'm inner")
    
    return nested_function  # function being returned here

# set the function outer_function() to a variable called inner_function
inner_function = outer_function()

# activate the function inner_function()
inner_function()
```

    I'm outer
    I'm inner
    

---
### Decorators

Let's imagine that you have a bunch of functions in your class or in your module, and you want to add some functionality to each of these functions. You can use a Decorator for this purpose. A Decorator function is a function that wraps another function and gives that function some additional functionality.

> `def decorator_function(function):
    def wrapper_function():
        # do something before
        function()  # you can also run function() multiple times
        # do something after
    return wrapper_function`
    
Let's say you want to create a simple function that prints a greeting, but with an added time delay.


```python
import time

def say_hello():
    time.sleep(2)  # time delay of 2 seconds
    print("Hello")
    
say_hello()  # prints with a 2 second delay
```

    Hello
    

Now let's imagine you wanted to add a time delay on several greetings. You would have to type in code to multiple places. This where the decorator comes in handy. Before we trigger the function that's passed in to the decorator function, we can add the delay. We can call the decorator in front of the function using the `@` sign.


```python
import time

def delay_decorator(function):
    def wrapper_function():
        time.sleep(2)
        function()
    return wrapper_function
```


```python
@delay_decorator
def say_hello():
    print("Hello")
    
@delay_decorator
def say_bye():
    print("Bye")

@delay_decorator
def ask():
    print("How are you?")

ask()  # prints with a 2 second delay
```

    How are you?
    

---
### Asynchronous Programming

Some operations take a long time, for e.g. web calls, network IO, complex data processing, etc. We don't want to stop everything just because one operation is taking a really long time. In synchronous programming, the methods are written to perform one task at a time. If a function depends on the other function's output, it has to wait to finish the execution of that function. Asynchronous programming also takes one execution at a time but the system may not wait to finish the execution to move on to the next step. It means the processor doesn't sit idle and the program will perform another task while the previous task hasn't finished and is still running elsewhere.  
Python offers multiple options for managing long running operations; we're going to focus on a common scenario - web requests.  
Since Python 3.4, there is the `asyncio` module that provides this capability.


```python
async def load_data(session, delay):
    async with session.get(f'https:httpbin.org/delay/{delay}') as resp:
        await resp.text()
```

Below is a demo of synchronous programming:


```python
from timeit import default_timer
import requests

def load_data(delay): # one parameter
    print(f'Starting {delay} second timer')
    # make get call
    text = requests.get(f'https://httpbin.org/delay/{delay}').text
    print(f'Completed {delay} second timer')
    return text

def run_demo():
    start_time = default_timer()
    
    two_data = load_data(2) # 2 second delay
    three_data = load_data(3) # 3 second delay
    
    elapsed_time = default_timer() - start_time
    print(f'The operation took {elapsed_time:.2} seconds')
    
def main():
        run_demo()
        
main()
```

The program started the 2 second delay, then it finished. Then, it started the 3 second delay, and finished. The operation took 7.8 seconds in total. The extra 2.8 seconds is how long it took to spin up and tear down the appropriate connections.

Now let's try to use asynchronous programming:


```python
from timeit import default_timer
import aiohttp
import asyncio

async def load_data(session, delay):
    print(f'Starting {delay} second timer')
    async with session.get(f'http://httpbin.org/delay/{delay}') as resp:
        text = await resp.text()
        print(f'Completed {delay} second timer')
        return text

async def main():
    # Start the timer
    start_time = default_timer()

    # Creating a single session
    async with aiohttp.ClientSession() as session:
        # Setup our tasks and get them running
        two_task = asyncio.create_task(load_data(session, 2))
        three_task = asyncio.create_task(load_data(session, 3))

        # Simulate other processing
        await asyncio.sleep(1)
        print('Doing other work')

        # Let's go get our values
        two_result = await two_task
        three_result = await three_task

        # Print our results
        elapsed_time = default_timer() - start_time
        print(f'The operation took {elapsed_time:.2} seconds')

asyncio.run(main())
```

We have created two tasks called `two_task` and `three_task`, using `create_task`. `load_data`, as the name suggests, loads data. Loading data takes a while and this is our task. While the data is being loaded, we can keep executing our code. Then, once you're ready to go get the answer of your task, you can grab that data using `await`. We have assigned the results to two variables. `await` is logically going to pause your code.  
`load_data` calls an endpoint that pauses for x number of seconds; 2 and 3 in this case. If the program was running synchronously, it would have taken 5 seconds. It takes less time in our case. We have a 3 second delay so it takes at least that much time.  
`async` is saying that in this construct, we're going to call `await`. Somebody else who is going to call this function can also await on whatever the operation is inside of here. If you're going to use `await`, it always has to be inside of an `async` construct.

---
### Modules and Packages
You've created some functions and now you want to reuse them in your current application or maybe other applications. The way to do that is by using modules, and you can import modules in separate projects using packages.  
A *module* is a Python file with functions, classes and other components, that are used to break down the code into reusable structures. Each module is responsible for a different bit of functionality in your program.

#### Creating a module
To create a module, all you need to do is to create a file and add in the appropriate code.


```python
# Let's name the below Python file `helpers.py`

# create a function named display
def display(message, is_warning=False):
    if is_warning:
        print('Warning!')
    print(message)
```

#### Using a module
To be able to use a module, we need to import it. Let's look at different methods of doing that. You can either simply import the module, or import everything inside the module and make it globally available, or import specific items from the module.


```python
# import module as namespace
import helpers
helpers.display('Not a warning')

# import all into current namespace
from helpers import *
display('Not a warning')

# import specific items into current namespace
from helpers import display
display('Not a warning')
```

#### Packages
*Packages* are published collections of modules. Through packages, you can easily use modules that other people have created. Through experience and just using Python, you'll get to know about the available packages.  To find out what packages are available, you should just do an Internet search. You can also see a list of available packages in the [Python Package Index](https://pypi.org/) (PyPI). If you're about to do something that somebody else has already done, you should always search for packages first, since someone else probably had that problem and solved it.

#### Installing packages
`pip` is the command line installer for python.  
You can install an individual package or a list of packages.


```python
# install and individual package
pip install <package_name>
```

In order to install a list of packages, set them up inside of a text file, called `requirements.txt`, which is nothing but a text file with a list of all the packages you want to install. By default, the most updated versions of the packages will be installed.


```python
# requirements.txt
package1
package2
package3
```


```python
# install from a list of packages
pip install -r requirements.txt
```

---
### Virtual Environments

By default, packages are installed globally. This means that it is going to be available for every application you'll be creating. Due to this, version management becomes a challenge. As a best practice, when you're setting up your application, is to do a local install, and this is done inside a virtual environment.  
A virtual environment is nothing but a folder that has all of the code you're going to need to run your application. It can be used to contain and manage package collections.

#### Creating virtual environments
Step 1: Make sure you install `virtualenv` globally.


```python
# install virtual environment
pip install virtualenv
```

Step 2: Create the environment.


```python
# windows system
python -m venv <folder_name>

# OSX / Linux (bash)
virtualenv <folder_name>
```

#### Using virtual environments
You'll need to activate the environment in order to use it.


```python
# Windows system

# cmd.exe
<folder_name>\Scripts\Activate.bat

# Powershell
<folder_name>\Scripts\Activate.ps1

# bash shell
# first . is the location of source code 
# typically do this from current directory
. ./<folder_name>/Scripts/activate


# OSX/Linux (bash)
<folder_name>/bin/activate
```

---
### Calling an API

#### Web Service
When developers want to share the functionality of a function but not the actual code in the program, they can place the function on a web server. A programmer with the address of that function on the web server and the required permissions can call the function. This is called a web service.

#### API
You can't call a function unless you know the function name and the required parameters. When you create a web service, you create an Application Programming Interface (API). The API defines the function names and parameters so others know how to call your function.

Suppose you're a developer who sign up on my web site, or buys a license for my software, and is provided a unique key. When you call my web service, you provide your unique key, and I am able to verify whether the key has been approved for calls to my web service. Thus, **keys** allow developers to track which users have permissions to use a web service.  
**Note: You should not put your API key in your code! It should not be visible to other people, otherwise somebody might use your key to call the API.**

#### HTTP
Hypertext Transfer Protocol (HTTP) is a standard protocol for sending messages across the web. There are two standard protocols we use for sending messages under HTTP. The API documentation usually mentions if you need a GET or POST call.
- GET
    - Pass values in query string only
        - Special characters must be "escaped"
        - Limited amount of data
- POST
    - Pass values in query string and body
        - No need to escape special characters if passed in body
        - Can pass large amounts of data, including images, in body
        
The **requests** library simplifies making a POST or GET call from Python code. All the parameters required to call the API are mentioned in the API documentation.  
`requests.post(address, http_headers, function_parameters, message_body)`

Learning how to call APIs unlocks functionality from developers and software companies all around the world, so it's very beneficial to know how to do that. Although API parameters and key requirements will vary, the documentation will provide all the information you need to call a specific API.

---
### JSON
JSON is a standard data format that is used to pass data back and forth and many web services return data as JSON.  
JSON contains key-value pairs. A key can also have subkeys that have their corresponding sub-values. A key can also have a list of values.  
There are various JSON linting tools on the Internet that can be used to format and prettify the JSON output and make it easier to read.  

First, import the JSON library.  
To retrieve the value from a, request the key name:  
>`{"key":"value"}`  
>Suppose the results from your API were passed on to a variable called `results`.  
>`"requestId":"234gt84-asde-29384ugd"`
>`print(results['requestId'])`

To request a value from a `{"key":{"subkey0":"subvalue0, subkey1":"subvalue1",...}}`, specify the key name and the subkey name:  
>`print(results['key']['subkey0']`

To retrieve a value from a `{"key":{[listvalue0:[value0, value1,...]], listvalue1:[value0, value1,...]],...]}}`, specify the keyname and index position of the value to retrieve:
> `print(results['key']['listvalue0'][0])`  

You can also use a loop to print out each item in the list.
> `for item in results['key']['listvalue0']:
    print(item)`

#### Creating JSON



```python
import json
```


You can use Python dictionaries to create `"key":"value"` JSON objects.


```python
# create a dictionary object
identity = {
    'alias': 'Batman',
    'first name': 'Bruce',
    'last name': 'Wayne',
}

# add additional key pairs as needed
identity['city'] = 'Gotham'

print(identity)
```

    {'alias': 'Batman', 'first name': 'Bruce', 'last name': 'Wayne', 'city': 'Gotham'}
    


```python
# convert dictionary to JSON object
identity_json = json.dumps(identity)
print(identity_json)
```

    {"alias": "Batman", "first name": "Bruce", "last name": "Wayne", "city": "Gotham"}
    

You can create nested dictionaries to create JSON in the format `{"key":{"subkey0":"subvalue0","subkey1":"subvalue1",...}}`.


```python
identity = {
    'alias': 'Batman',
    'first name': 'Bruce',
    'last name': 'Wayne',
    'city': 'Gotham'
}

# create an empty dictionary
role = {}

# add a key to the role dictionary
# and assign it to identity dictionary
role['dark knight'] = identity

# convert dictionary to JSON object
role_json = json.dumps(role)
print(role_json)
```

    {"dark knight": {"alias": "Batman", "first name": "Bruce", "last name": "Wayne", "city": "Gotham"}}
    

You can add lists to dictionaries to create JSON in the format `{"key":{[listvalue0:[value0, value1,...]], listvalue1:[value0, value1,...]],...]}}`.


```python
# create a list of enemies
enemies_list = ['Joker', 'Riddler', 'Bane',
         'Two-Face', 'Scarecrow', 'Penguin']

# add list to dictionary
identity['enemies'] = enemies_list

# convert dictionary to JSON object
identity_json = json.dumps(identity)
print(identity_json)
```

    {"alias": "Batman", "first name": "Bruce", "last name": "Wayne", "city": "Gotham", "enemies": ["Joker", "Riddler", "Bane", "Two-Face", "Scarecrow", "Penguin"]}
    

When creating and reading JSON
- use print statements to help you debug.
- use a JSON linting tool to make the JSON easier to read.
- have a print out of the full JSON so you can figure out the structure when reading specific elements.

---
