# Magic Parser vs. SimpleXML


Recently a client hired me to create a PHP script to populate a MySQL database
based on data in a series of XML files. A simple enough task, but the client
had recently spent $80 on a product called Magic Parser and so wanted me to
use the tool in my code. While I am sure the expenditure would have been worth
it had the task been different, I was not at all impressed with Magic Parser's
performance with this particular task.

When I'm parsing through XML code I really want a tree-like structure. Magic
Parser returns everything in a flat array, with tag names separated by
slashes. For example:
`
<foo>
    <bar>A</bar>
<foo>
`

would return
`
Array([FOO]=>"", [FOO/BAR]=>"A")
`

This causes problems if there is more than one tag with the same name. For
example:
`
<foo>
    <bar>A</bar>
    <bar>B</bar>
<foo>
`

They work around this by sending three arrays to the callback function:
`
Array([BAR]=>"A")
Array([BAR]=>"B")
Array([FOO]=>"", [FOO/BAR]=>"A")
`

This actually makes it harder to process the XML than if I were using
SimpleXML, as SimpleXML would return:
`
SimpleXML Object([bar]=>Array([0]=>"A", [1]=>"B"))
`

In SimpleXML I would just have to cycle through the array attached to the
"bar" property of the SimpleXML object. With MagicParser I have to figure out
if it's sending me a "BAR" array or a "FOO/BAR" array.

The moral of the story is that if you just want to parse XML data, give Magic
Parser a wide berth and stick with something like SimpleXML. It does the job
better, it's free, and it's part of PHP's default installation.

