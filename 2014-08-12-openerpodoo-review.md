# OpenERP&#47;Odoo Review


## Background

  
This post is meant to be a helpful reference for developers who are either
just beginning OpenERP development or who are considering beginning OpenERP
development. It's also a fairly thorough catalog of my gripes with it.
Hopefully this will help save someone somewhere some pain. I start with a
summarized list of its pros and cons, followed by a more thorough explanation
of each of the cons, and finish up with a list of tips that I personally would
have found useful back when I was getting started.

I've been working with OpenERP v7 since December 2013. (Version 8 has since
been released as "Odoo.") In the past 9 months I've gone from being a complete
newbie to being familiar and competent with every aspect of this ERP
framework. I have no basis for comparing it to other ERP solutions, as it's
the first I've ever worked with, but I can make a decent comparison between it
and other Python-based projects.

## Pros &amp; Cons

  
Pros:

  * Python-based.
  * Heavy emphasis on open source code.
  * Modular design with a user-friendly "app-store" interface for non-developers.
  

  
Cons:

  * Not very idiomatic.
  * XML gets used for _everything._
  * The inheritance system is painful.
  * The module system can be unreliable.
  * The ORM doesn't follow the ActiveRecord pattern.
  * Poor documentation.
  

  
Before I launch into describing the pain points in more detail, I should first
acknowledge that OpenERP began in 2005 as TinyERP. Had I myself begun an ERP
system in 2005, I might not have done much better. The ecosystem was not the
same back then, and a lot of my gripes likely are the result of the hindsight
that the original developers didn't have. That said, there _are_ a lot of
things about OpenERP that tripped me up and that _still_ trip me up from time
to time. If you're just getting started with OpenERP, learn a little from my
experience and read on. (And if you have the chance to try
[Tryton](http://www.tryton.org/) instead, give it some consideration. From the
outside, it appears to be a bit more Pythonic than OpenERP.)

### Not very idiomatic

  
The OpenERP developers seem to have built custom systems for _everything_. The
client whose OpenERP system I was building on had advertised for a "Python
developer," but I felt like almost _none_ of my Python knowledge got leveraged
because almost _everything_ had been reinvented in one way or another. The
"workflows" system could have been done with
[blinker](https://pypi.python.org/pypi/blinker), but instead they opted to
implement a custom, hybrid Python/XML-based system. The templating system
could have been implemented quite nicely with
[Mako](http://www.makotemplates.org/), but instead they opted to implement yet
_another_ custom, hybrid Python/XML-based system. Speaking of which...

### XML gets used for _everything_

  
Which wouldn't be so bad if I didn't hate XML so much. But in my opinion, XML
is a poor match for Python. As I understand it, XML is great for enforcing
correctness and catching errors early. It's especially great when you're
working with a static-typed language, from what I hear. And while OpenERP
_does_ catch template errors upon module installation, the error messages
aren't especially intuitive. On top of this, a bad template with literally
break the entire server. If your object models doesn't match their templates
somehow, the server will lock everyone out and answer every login attempt with
a stack trace.

### The inheritance system is painful

  
Rather than use Python's package system, OpenERP uses a custom package system.
"Add-on paths" are defined in the server's configuration file in order of
priority, and modules within these add-on paths can import each other and
override each other's code.

It's a fairly flexible system, but it makes tracing code to find bugs more
complicated than it has to be. In a normal Python project, figuring out what
code is executing is as simple as looking at the imports at the top of the
file and following them all the way up. With OpenERP, you still have normal
Python imports to worry about, but now you also have OpenERP's system to worry
about as well. In OpenERP's system, module imports are defined as strings in
an array in __openerp__.py, with their order in the array determining in which
order they get imported. This in turn determines which version of, say,
"res.company" your module's "res.company" class inherits from. Because in
OpenERP, class definition and inheritance are determined by magic class
properties. If the __name__ string is set, the class is being defined, which
_can_ lead to occlusion if the name is used in a previously imported module.
If the __inherit__ string is set, then the class inherits from a class
belonging to the last-imported module whose __inherit__ or __name__ property
matches.

This system is, in my opinion, needlessly complicated and unidiomatic. I could
be wrong, as I've never written an ERP myself, but I suspect simply allowing
developers to leverage their existing knowledge from, say,
[SQLAlchemy](http://www.sqlalchemy.org/) might have been a better design
decision.

### The module system can be unreliable

  
Upgrading the database after changing the codebase doesn't always work the way
you would expect it to, for example. This has led to fun situations where, for
example, a module works on my dev box but not on the test box because _once_
the module had defined a column on one of its models, and that column wasn't
removed from my dev database during an upgrade and some _other_ package turns
out to have relied on that column being in the database.

Installing a new module can likewise be treacherous. I once spent hours trying
to figure out why the new menu items I had added to a module weren't
appearing, despite my settings in the security permissions CSV being correct,
and despite the menu items showing up under the menu items tree in the user
interface settings section of OpenERP. There were no errors whatsoever during
installation, and there were no errors from the module during runtime. In the
end, it turned out that one of the modules in its dependency chain was having
trouble connecting to a remote server, which somehow led to the new menu items
not being displayed. I understand that managing actors that are able to
unilaterally alter each other's states is a very difficult problem. I
understand that mutating shared, global state is going to happen within the
context of an ERP. But I can't help but feel it could be done a whole lot
better.

### The ORM doesn't follow the ActiveRecord pattern

  
I know there's some dissent about ActiveRecord's usefulness, but it _is_ a
very common pattern at this point. OpenERP instead uses a very restricted,
custom ORM that requires passing around cursor handles, user IDs, object IDs,
and contexts. Because of how restricted the ORM is, very few of the
efficiencies that can be realized by eschewing an ActiveRecord pattern are
realized, and you get none of ActiveRecord's idiomatic syntax.

### Poor documentation

  
Don't get me wrong: there's a lot of it out there, and the fact that there
_is_ documentation at all puts them ahead of a lot of projects. The problem is
primarily that it's not organized well. There isn't a clean division between
documentation that would primarily appeal to developers and business users.
The style tends to veer toward verbose tutorials rather than plain API
listings. There are holes here and there, where I've been forced to just dig
into the source code to figure out how things work. There are different
versions of the documentation for different versions of OpenERP, each with
unique holes, which leads to jumping between versions in order to fill in the
holes and hoping that what held true for version 5.0 still holds true for
version 7.0.

And perhaps worst of all, there are the blank pages and dead links that
started popping up everywhere after the developers rebranded OpenERP as Odoo.
I use [StartPage](https://startpage.com/) and [IxQuick](https://ixquick.com/)
to search for OpenERP documentation because they tend to work better than the
search on the OpenERP/Odoo website. But they're not working quite as well as
they used to, because oftentimes about half the results in the top five will
be either to a thread on their old forum or to a page that appears to have
literally just had its content removed. The latter case is particularly
frustrating: the heading is still there, the page is still in the table of
contents for that version of OpenERP's documentation, but there is no longer
any content.

## Tips

  
If this hasn't scared you away from OpenERP, here are a couple pointers that I
wish someone had given me when I first got started:

  * Don't ignore warnings or errors, no matter which module they're coming from. It's usually safe to assume that every module in OpenERP is reliant on every other module. One module that isn't functioning _exactly perfectly_ can very well cause mysterious errors in a seemingly unrelated one.
  * Backup your database before you start uninstalling and re-installing modules. You will want it when the database migrations fail and your data ends up in a hopelessly impossible state, or when everything breaks because you've violated another module's assumptions about the database model you've been changing.
  * Security CSVs. Learn how to use them. Don't forget about them. They will likely be the reason none of your new menu items and models are showing up for non-admin users when you're first getting started.
  

  
Best of luck in your OpenERP/Odoo adventures! Let me know your thoughts and
opinions in the comments, and feel free to ask questions. I'll do my best to
answer them.

