# Object oriented programming in Python

Interfaces are not really a thing in Python, but you can sort of implement them using the `__init__subclass__(cls, ..)` ([PEP-487](https://peps.python.org/pep-0487/)) method.

## The registry pattern

A cool use-case for this is the [registry pattern](https://stackoverflow.com/questions/57626912/registry-pattern-with-init-subclass-and-sub-classable-registry), which allows you to register subclasses as they are defined.

i.e.:

```python
_REGISTRY = {}

class RegisteredClass():
    reference: str

    def __init_subclass__(cls, **kwargs):

        # Run some validations (i.e. on property data types)

        super().__init_subclass__(**kwargs)
        _REGISTRY[cls.reference] = cls


class SomeRegisteredClass(RegisteredClass):
    reference = "some_registered_class"


def get_registered_class(reference: str) -> RegisteredClass:
    """Returns instance of a RegisteredClass by it's reference"""
    return _REGISTRY[reference]()
```

### Why would we want this?

This pattern can be useful for Python repos where you want other users to be able to write plugins (e.g. new data connectors), and make sure the plugins behave in a way you want them to (through your validators in `__init_subclass__`).

### Why not Abstract Base Classes?

Abstract base classes don't fail fast, they only fail as they are used. `__init_subclass__` runs as a subclass definition is read in runtime (i.e. when the module of that subclass gets imported).
