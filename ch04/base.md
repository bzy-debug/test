Tests for WBE Chapter 4
=======================

Chapter 4 (Constructing a Document Tree) adds support for the document tree
(i.e. the DOM).  This file contains tests for the additional functionality.

    >>> import wbemocks
    >>> import browser

Notes
=====

You'll need to modify the `__repr__` method on `Element` to print the
attributes in each element. The method should now have this
definition:

```
class Element:
    def __repr__(self):
        attrs = [" " + k + "=\"" + v + "\"" for k, v  in self.attributes.items()]
        attr_str = ""
        for attr in attrs:
            attr_str += attr
        return "<" + self.tag + attr_str + ">"
```

Testing HTMLParser
==================

`HTMLParser` is a class whose constructor takes HTML body text as an argument, and
can parse it. We can test it by parsing a document and printing the
resulting tree.

    >>> def test_parse(text):
    ...     parser = browser.HTMLParser(text)
    ...     browser.print_tree(parser.parse())

The implicit ``html` and `body` (and `head` when needed) tags are added:

        >>> test_parse("<html><body>test</body></html>")
         <html>
           <body>
             'test'

Quotation marks around `Text` objects should be different depending on if
there are quotes within:

        >>> test_parse("<html><body>baker's dozen</body></html>")
         <html>
           <body>
             "baker's dozen"

Missing tags are added in:

        >>> test_parse("test")
         <html>
           <body>
             'test'

        >>> test_parse("<body>test")
         <html>
           <body>
             'test'

Head tags are put in the head, and other tags, such as `div`, are put
in the body. Also, tags such as `base` are self-closing:

        >>> test_parse("<base><basefont></basefont><title></title><div></div>")
         <html>
           <head>
             <base>
             <basefont>
             <title>
           <body>
             <div>

Missing end tags are added:

        >>> test_parse("<div>text")
         <html>
           <body>
             <div>
               'text'

Attributes can be set on tags:

        >>> test_parse("<div name1=value1 name2=value2>text</div")
         <html>
           <body>
             <div name1="value1" name2="value2">
               'text'

Testing Layout
==============

First, let's test that basic layout works as expected:

    >>> parser = browser.HTMLParser("<p>text</p>")
    >>> tree = parser.parse()
    >>> lo = browser.Layout(tree)
    >>> lo.display_list
    [(13, 20.25, 'text', Font size=12 weight=normal slant=roman style=None)]

Moreover, layout should work even if we don't use the
explicitly-supported tags like `p`:

    >>> parser = browser.HTMLParser("<div>text</div>")
    >>> tree = parser.parse()
    >>> lo = browser.Layout(tree)
    >>> lo.display_list
    [(13, 20.25, 'text', Font size=12 weight=normal slant=roman style=None)]


Parse HTML with italic text

    >>> parser = browser.HTMLParser("<i>Italic</i> Normal.")
    >>> tree = parser.parse()

Create a Layout from the parsed tree

    >>> lo = browser.Layout(tree)
    >>> wbemocks.print_list(lo.display_list)
    (13, 20.25, 'Italic', Font size=12 weight=normal slant=italic style=None)
    (97, 20.25, 'Normal.', Font size=12 weight=normal slant=roman style=None)
