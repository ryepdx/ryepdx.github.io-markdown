# Typed Python with MyPy


[The "The DAO" hack](http://www.coindesk.com/understanding-dao-hack-
journalists/) threw me down a "how safely can I write code" rabbit hole, which
has led to me picking up typed Python for [my current project](https://cpay.us).

Support for "type hints" was introduced in [PEP
484](https://www.python.org/dev/peps/pep-0484/) and implemented for the first
time in Python 3.5. As Python is not and will not at any time in the future be a
proper statically typed language, the PEP describes a method of "hinting" at
types using structured comments and other syntactic sugar which the Python 3.5
compiler studiously ignores. External utilities can then interpret and enforce
as they will. While in theory this allows for competition among approaches, at
present there's only one functional type checker:
[MyPy](http://mypy.readthedocs.io/en/latest/).

On the whole, I've very much enjoyed writing typed Python. It's corrected one of
the Python pain points I've long chafed against, which is the lack of parse-time
argument type checking. Duck typing can be nice, but... okay, it's really not
all that nice. By taking the time to define the classes I want up front and
define their behavior in unit tests, I'm able to write safer, cleaner code. By
defining the types of parameters, I lose the risk of swapping them or leaving
them out somewhere.

It's an obvious point, but having better support for static code analysis in
Python is really very nice.

The downside, of course, is that the third party modules you'll be working with
probably won't be written in typed Python. After all, PEP 484 is only available
in the latest version of Python 3. At that point you're left with the option of
either stubbing out the module in a way that's intelligible to MyPy, or telling
MyPy to ignore the module and lose the safety of type checking in potentially
large chunks of your code. So in that sense, writing in typed Python feels like
working on a rocket strapped to a tricycle.

But at least it's a pretty tricycle.
