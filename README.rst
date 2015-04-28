====================
Chainable Validators
====================

This package contains a set of simple functions to facilitate creation of
chainable validator methods for developing data validation schemata.

Although it provides a few validation function, it's main purpose is
extensibility.

Installing
==========

The PyPI package name is ``chainable-validators``. You can install the package
using ``pip`` or ``easy_install``::

    pip install chainable-validators

    easy_install chainable-validators

Basic concepts
==============

The basis of all validation are chainable validators (validation functions) and
the validation chains which are created using ``make_chain()`` calls.

The chainable validators are found in the ``validators.validators`` module. For
convenience, they can be imported directly from the ``validators`` package::

    >>> from validators import required, istype, gte

The chainable validators can be used stand-alone or as part of a chain. For
standalone usage, pass the value to the validator. ::

    >>> required(None)
    Traceback (most recent call last):
    ...
    ValueError: required value missing

Some validators are parametric. They are first invoked with a parameter, and
the return value is then used as a chainable validator. ::

    >>> isint = istype(int)
    >>> isint('foo')
    Traceback (most recent call last):
    ...
    ValueError: not int

To build a chain of desired validators, we first compile a list of chainable
validators::

    >>> from validators import make_chain
    >>> fns = [required, istype(int), gte(2)]
    >>> chain = make_chain(fns)
    >>> chain(None)
    Traceback (most recent call last):
    ...
    ValueError: required value missing
    >>> chain('1')
    Traceback (most recent call last):
    ...
    ValueError: not int
    >>> chain(1)
    Traceback (most recent call last):
    ...
    ValueError: value too small
    >>> chain(3)
    3

The validators in the chain are invoked in order until the last one is called,
or ``ValueError`` or ``validators.ReturnEarly`` is raised. When no exceptions
are raised, the return value of the last chainable validator is returned. In
case ``ReturnEarly`` is raised, it is not propagated to the chain's caller, but
instead the original value is returned.

List of built-in validators
===========================

The following is a list of built-in vaidators.

Validators can be parametric or simple. Simple validators can be used directly.
Parametric validators take an argument and return a chainable validator
functions.

In the list below, if you encounter a validator that looks like ``foo(bar)``,
it's a parametric validator.

- ``optional(default=None)`` - interrupts validation chain if value is None or
  a user-supplied default value
- ``required`` - rejects None
- ``nonemtpy`` - rejects empty sequences (string, list, dict)
- ``boolean`` - reject non-boolean values (other than True, False, 1, and 0)
- ``istype(t)`` - rejects values that are not of type ``t``
- ``isin(collection)`` - rejects values that are not in ``collection``
  (collection is a sequence such as string, list, or dict)
- ``gte(num)`` - rejects values that are not greater than or equal to ``num``
- ``lte(num)`` - rejects values that are not less than or equal to ``num``
- ``match(regex)`` - rejects values that do not match the ``regex`` object
  (``regex`` object is a valid ``re.RegExp`` instance or object with a
  ``match()`` method)
- ``url`` - rejects values that are not URLs

Writing your own validators
===========================

It is possible to write your own validators. To write a simple chainable
validator, use the ``validators.chain.chainable`` decorator. ::

    >>> from validators import chainable
    >>> @chainable
    ... def my_validator(s):
    ...     if not s.startswith('foo'):
    ...         raise ValueError('does not start with foo')
    ...     return s
    ... 
    >>> my_validator('foobar')
    'foobar'
    >>> my_validator('barfoo')
    Traceback (most recent call last):
    ...
    ValueError: does not start with foo

To write a parametric validator, define the chainable validator in a closure::

    >>> def my_parametric(start):
    ...     @chainable
    ...     def validator(s):
    ...         if not s.startswith(start):
    ...             raise ValueError('does not sart with {}'.format(start))
    ...         return s
    ...     return validator
    ... 
    >>> validator = my_parametric('baz')
    >>> validator('bazfoo')
    'bazfoo'
    >>> validator('foo')
    Traceback (most recent call last):
    ...
    ValueError: does not sart with baz

Now you can use these validators in chains like other validators.

Reporting bugs
==============

Please report any bugs or feature requests to the `issue tracker`_.

.. _issue tracker: https://github.com/Outernet-Project/chainable-validators/issues