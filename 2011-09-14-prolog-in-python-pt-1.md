# Prolog in Python (pt. 1)


I've recently become enamored with the idea of querying a Prolog program with
a MySQL database backend from a Python script. Specifically I want to create a
Django web app that will allow a user to ask for the price of a thing in terms
of anything else. Prolog seems like an excellent choice for the value-finding
engine as value-finding in this case will mostly consist of finding a path of
trade between two things. For example, in order to find the price of apples in
terms of bananas, the value finding engine will first query its database to
see if it already knows that off-hand. If it doesn't, it then has to see what
it does know about the price of apples and then see if anything it can trade
apples for can then be traded for bananas. It has to keep branching out until
it finally hits bananas.

I was able to complete a proof-of-concept value engine [with just 9 lines of
Prolog](https://github.com/ryepdx/prolog-appraise/blob/master/appraise.pl),
minus comments.

The next step, of course, was to find a way to allow Python to call Prolog
predicates. As I was using SWI-Prolog, I downloaded
[PySWIP](https://code.google.com/p/pyswip/).

[PySWIP](https://code.google.com/p/pyswip/) is a simple gateway between the
SWI-Prolog interpreter and Python. The project was briefly orphaned for a
while and the project's author is actively looking for another developer to
take the reins, so I can't recommend this as an enterprise-grade solution.
However, it seemed that for my purposes it would work just fine.

PySWIP did not work right out of the box. I had SWI-Prolog 5.10.4 installed on
my machine and the latest version that PySWIP supported at the time was
5.6.34. Running

`from pyswip import Prolog`

resulted in

`libpl (shared) not found. Possible reasons:  
1) SWI-Prolog not installed as a shared library. Install SWI-Prolog (5.6.34
works just fine)`

Fortunately the fix was easy. After some Googling I surmised that the issue
stemmed from the fact that libpl.dll had been renamed to swipl.dll in the
versions after 5.6.34. Navigating to C:\Program Files\pl\bin and making a copy
of swipl.dll named libpl.dll did the trick.

Here's a little example of using an external Prolog program in Python.

`from pyswip import Prolog`

`prolog = Prolog()`

`prolog.consult("appraise.pl")`

`# A couple preliminary assertions.  
prolog.assertz("value(banana, 1 rdiv 4, bitcoin)")  
prolog.assertz("value(bitcoin, 1 rdiv 30, namecoin)")  
prolog.assertz("value(apple, 3 rdiv 4, banana)")  
prolog.assertz("value(orange, 1 rdiv 5, apple)")`

`# Now, how much is an orange worth in bitcoins?  
result = prolog.query("appraise_float(orange, Price, bitcoin)").next()  
print "An orange is worth %s bitcoins." % result["Price"]`

`# And how much is a bitcoin worth in apples?  
result = prolog.query("appraise_float(bitcoin, Price, apple)").next()  
print "A bitcoin is worth %s apples." % result["Price"]`

Running this results in  
`  
An orange is worth 0.0375 bitcoins.  
A bitcoin is worth 5.33333333333 apples.  
`

Next up: [SWI-Prolog/MySQL integration](http://ryepdx.com/2012/02/prolog-in-
python-pt-2/).

