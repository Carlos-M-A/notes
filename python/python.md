# Python

Python basic notes

Other good notes and cheatsheets I found:

## PYTHON PROJECT STRUCTURE

[Helpful doc](https://realpython.com/python-application-layouts/)


## METHODS AND ATTRIBUTES USE (instance, class and static ones)

```python
class MyClass:
    class_variable_1 = ''
    class_variable_2 = ''

    def __init__(self, var1, var2):
        self.instance_variable_1 = var1
        self.instance_variable_2 = var2

    def __str__(self):
        return str(self.instance_variable_1)

    @classmethod
    def public_class_method(cls):
        #accessing class variable
        cls.class_variable_1
        #accessing class method
        cls.public_class_method()

    @staticmethod
    def public_static_mehotd():
        #No class varriable or method should be accessed from a static mehod
        #But if it is needed. you can:
        #accessing class variable
        MyClass.class_variable_1
         #accessing class method
        MyClass.public_class_method()

    def public_instance_method(self):
        # accessing class variable to read/write (best option)
        MyClass.class_variable_1
        
        # accessing class variable to read/write (other options)
        type(self).class_variable_1
        self.__class__.class_variable_1

        # accessing ONLY for reading (other option)
        self.class_variable_1

        # Doing this, class_variable wiil be overrided, 
        # by a new instance variable with the same name
        self.class_variable_1 = 1

        # accesssing instance variable
        self.instance_variable_1

        # calling class or static method
        MyClass.public_class_method()
        self.public_class_method()
        self.__class__.public_class_method()

        # calling instance method
        self._private_instance_mehod()
    
    def _private_instance_method(self):
        self.instance_variable_1
```

## METHODS ORDER IN CLASSES

Methods order:
1. `__init__()`
2. public methods
3. private methods
4. magic methods

```python
class SomeClass(object):

    def __init__(self):
        "__init__() always the first"

    ########################
    #    Public Methods    #
    ########################
    def a_method(self):
        "public methods, in order of importance"

    ########################
    #   Private Methods    #
    ########################
    def _private_method(self):
        "then private methods, grouped by importance or related function"

    ########################
    #     Magic Methods    #
    ########################
    def __magic_methods__(self):
        "magic methods last"

```