# Metaclass

## Introduction
reference: https://realpython.com/python-metaclasses/  

(for new-style python class inherit from object like: ***class A(object): pass***)  
type is an instance of type.  
class is an instance of type.  
ordinary-object is an instance of class.  
that makes type as an metaclass.  so: metaclass used to customize classes' behaviour(instantiation).  Metaclasses are sometimes referred to as ***class factories***  

we can use ***type(name, bases, dict)*** to dynamic creates a class named name with ***__bases__*** eq to bases and ***__dict__*** eq to dict  

exp:  
    
    Foo = type('Foo', (object), {attr=10, func=lambda obj: obj})

what happend during statement: ***f = Foo()***?  
1. the ***__call__*** method is invoked. while the Foo is new-style class, its parent calss ***type***(the metaclass)'s ***__call__*** method is invoked.  
2. that ***__call__*** method in turn invokes the ***__new__*** method and ***__init__*** method.  

a class devired from ***type*** makes it as a metaclass.  
exp:  

    class MyMeta(type):  
        def __new__(cls, name, bases, dct):  
            x = super().__new__(cls, name, bases, dct)  
            x.attr = 100  
            return x  

    class MyClass(metaclass=MyMeta):  
        pass;


## Source code review

how to implement metaclass? understanding metaclass by code.  


