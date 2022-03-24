# fastai "docment" Sprint Style Guide

## Introduction

This is a doc-sprint specific style guide, providing you with direction on how to properly "docment" and add type hints to methods and functions. For everything else you should follow [the style documentation](https://docs.fast.ai/dev/style.html).

## Typing & Type Hints

All public method and function arguments and return types should have type hints, whenever a reasonably simple and useful type hint can be used accurately.

A type which is incorrect or more restrictive than necessary can cause IDEs to incorrectly flag passed arguments as errors. In this case, it is preferred for the type hint to be less restrictive or to be removed entirely.

Type hints can be added to private methods and functions, but they must enhance the existing context. Type hints are an additional item to be maintained, so don't add them to private methods and functions unless they're going to be helpful to other developers.

One noteable exception for adding type hints is if the return type is `self`. Then the type hint should be skipped.

### Python 3.10 type hints

Rather than using `typing` methods such as `Union` or `Optional`, the convention is to specify *and* or *or* using Python 3.10 style type hints. This is a practical design choice, aiming for less key-strokes while still being readable.

- `TypeA|TypeB` denoting `TypeA` **or** `TypeB`.
- `TypeA|TypeB|None` denoting that `TypeA` and `TypeB` are optional

This requires adding:
```python
from __future__ import annotations
```
to the imports in all notebooks with `|` to backport Python 3.10 style type hints to 3.7-3.9.

### Generic Types

Generic types should be added where they provide clarity to the type hint without being overly verbose or complicated. But they must be correct.

A generic type which is incorrect or more restrictive than necessary can cause IDEs to incorrectly flag passed arguments as errors. In this case, it is preferred for the type hint to be less restrictive or not contain a type at all.

### Custom Type Aliases

To simplify type hinting, fastai has some custom type aliases. 

Most fastai methods accept an iterator, a sequence, or a collection. `listy` is type alias which combines them all:

```python
T = TypeVar('T')
listy = Union[Iterable[T], L, fastuple]
```

along with fastcore classes `L` and `fastuple`.

It is possible to add generic types to `listy`.

```python
listy[int]
```

Where the input expects a single or list-like of items, use `listified`:

```python
T = TypeVar('T')
listified = Union[T, Iterable[T], L, fastuple]
```
with the generic type hint required:

```python
listified[int]
```

### List and Tuple

If a list or tuple is needed instead of `listy`, use `list` and `tuple` directly instead of `typing.List` or `typing.Tuple`:

* `list[int]`
* `tuple[int,int]`
* `tuple[int,...]`

`tuple` type hints require explicit dimensions to be specified or the use of `type,...` for variable dimensions, while `list` type hints only need to specify the type of the list for variable dimensions.

### When to use the `typing` package?

`typing` methods are used when needed. Currently fastai used the following a handful of `typing` methods, although more can be added as needed: 

* `Callable`
* `Any`

`Callable` type hints should define the return type(s) of the function. If the function input varies or is consistently long (ex: `opt_func`) then use `...` to denote variable input types:

```python
class Learner(GetAttr):
    def __init__(self,
    	...,
    	opt_func:Callable[..., Optimizer|OptimWrapper]=Adam,
    	splitter:Callable[nn.Module, list[nn.Parameter]]=trainable_params,
    	...,
    ):
```
but if it is short and known (ex: `splitter`), then add the input type to `Callable`. Methods forgo a `Callable` return type.

### Checking type hints for errors

Type hints should be validated to be error free by setting a type checker, such as Pylance (VSCode), Pyright, or mypy to basic type hint checking (or equivalent).

Due to quirks of each individual type hint checker—such as runtime functionally being unparsable pre-runtime (such as fastcore’s `mk_class`) or calling methods which lack type hints—not all type hint errors will be resolvable. 

A reasonable effort should be made to identify and correct all other type hint errors.

## Documenting methods and classes

Typically you want to aim for one-line docstrings unless it is *absolutely* necessary.

Along with this, you combine more explainability into a function through [docments](https://fastcore.fast.ai/docments). These are comments next to each input variable allowing for us to describe how each variable should be written. Docments should be concise and specific.

As an example, here is a "well documented" function definition:

```python
def addition(
    a:Number, # The first number to be added
    b:Number, # The second number to be added
) -> Number:
    "Adds two numbers together"
    return a+b
```

Very clearly you can see that even though the docstring is quite small, when combined with the docments we get a very clear return.

Combine this with the `show_doc` functionality in `nbdev`, and you are presented with a clear documentation snippet:

```python
>>> from nbdev.showdoc import show_doc

>>> show_doc(addition)
```

<h4 id="addition" class="doc_header"><code>addition</code><a href="__main__.py#L1" class="source_link" style="float:right">[source]</a></h4>

> <code>addition</code>(**`a`**:`Number`, **`b`**:`Number`)

Adds two numbers together

|             | Type     | Default | Details                       |
| ----------- | -------- | ------- | ----------------------------- |
| **`a`**     | `Number` |         | The first number to be added  |
| **`b`**     | `Number` |         | The second number to be added |
| **Returns** | `Number` |         |                               |

If the return value requires additional documentation, we can add a return docment too:

```python
def addition(
    a:Number, # The first number to be added
    b:Number, # The second number to be added
) -> Number: # The sum of `a` and `b`
    "Adds two numbers together"
    return a+b
```

Docments may be skipped for simple methods with one argument, provided the argument’s role is understandable from the doc string:

```python
def add_one(self, a:listy[Number]) -> list[Number]:
    "Adds one to all items in `a`"
    return [i+1 for i in a]
```

This will often be applicable for simple class methods.

Whether on a newline or one-liner, a method return type should always have a space after the closing parenthesis and the return arrow: `) -> list[Number]:`

## Additional Style Examples

### Docmenting a Class method

When docmenting a class method with `self` or `cls`, keep self on the method definition. Types are placed before the default values, if applicable:

```python
class Learner:
    ...

    def fit(self, 
        n_epoch:int, # Number of training epochs
        lr:float|None=None, # Learning rate, defaults to `self.lr`
        wd:float|None=None, # Weight decay, defaults to `self.wd`
        cbs:listified[Callback]|None=None, # Temporary `Callback` applied during training
        reset_opt:bool=False # Recreate optimizer before training
    ):
        "Fit `self.model` for `n_epoch` using `cbs`. Optionally `reset_opt`."
```

### Returning itself

If a method returns itself, such as a transform patch, there is no need to add the return type:

```python
@patch
def flip_lr(x:Image.Image):
    "Flips a pillow image"
    return x.transpose(Image.FLIP_LEFT_RIGHT)
```

### Miscellaneous

If an optional argument has a default set in code, concisely mention it in the docment (ex: `lr` and `wd`). If it isn’t used, there is no need to state it is “optional” as that is implied by the type hint of `None` (ex: `cbs`).

```python
class Learner:
    ...

    def fit(self, 
        n_epoch:int, # Number of training epochs
        lr:float|None=None, # Learning rate, defaults to `self.lr`
        wd:float|None=None, # Weight decay, defaults to `self.wd`
        cbs:listified[Callback]|None=None, # Temporary `Callback` applied during training
        reset_opt:bool=False # Recreate optimizer before training
    ):
        "Fit `self.model` for `n_epoch` using `cbs`. Optionally `reset_opt`."
```

Likewise, forego starting a docment with “list of”, as that is implied by the type hint of `list` (ex: `cbs`).

Boolean docments should be written to describe what the argument does if true  (ex: `reset_opt`). Forgo phrases such as “whether to,” “if set”, “if true”.

Keyword arguments or arguments (`**kwags` or `*args`) should not be docmented. In a notebook, internal keyword arguments should be viewable using `delegates` and `show_doc`.

### Incorrect Format Examples

When docmenting a class method, the result should not look like this:

```python
class Learner:
    ...

    def fit(self, 
            n_epoch:int, # Number of training epochs
            lr:float|None=None, # Learning rate, defaults to `self.lr`
            wd:float|None=None, # Weight decay, defaults to `self.wd`
            cbs:listified[Callback]|None=None, # Temporary `Callback` applied during training
            reset_opt:bool=False # Recreate optimizer before training
    ):
        "Fit `self.model` for `n_epoch` using `cbs`. Optionally `reset_opt`."
```

which might be due to how your IDE likes to tab on a newline.

Likewise this:

```python
class Learner:
    ...

    def fit(self, 
        n_epoch:int, # Number of training epochs
        lr:float|None=None, # Learning rate, defaults to `self.lr`
        wd:float|None=None, # Weight decay, defaults to `self.wd`
        cbs:listified[Callback]|None=None, # Temporary `Callback` applied during training
        reset_opt:bool=False # Recreate optimizer before training
):
        "Fit `self.model` for `n_epoch` using `cbs`. Optionally `reset_opt`."
```

and this:

```python
class Learner:
    ...

    def fit(self, 
        n_epoch:int, # Number of training epochs
        lr:float|None=None, # Learning rate, defaults to `self.lr`
        wd:float|None=None, # Weight decay, defaults to `self.wd`
        cbs:listified[Callback]|None=None, # Temporary `Callback` applied during training
        reset_opt:bool=False # Recreate optimizer before training
           ):
        "Fit `self.model` for `n_epoch` using `cbs`. Optionally `reset_opt`."
```

are also incorrectly formatted.

## Notes

* Arguments should have one indent relative to method definition.
* Only `cls` and `self` should be on same line of method name and first argument of `@patch` like `x` on `dihedral` sample.
* Closing parenthesis should be aligned with `def`.
* Keyword arguments (`**kwags`) should not be docmented.
