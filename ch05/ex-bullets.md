Tests for WBE Chapter 5 Exercise `Bullets`
==========================================

Add bullets to list items, which in HTML are `<li>` tags. You can make
them little squares, located to the left of the list item itself. Also
indent `<li>` elements so the text inside the element is to the right
of the bullet point.

Specifically, each `li` element should be indented by `2 * HSTEP`. The
bullet itself should be a black rectangle, 4 by 4 pixels, centered
vertically on the whole line (including leading) and with a horizontal
center one `HSTEP` from the start of the line. The `DrawRect` call
should in the `paint` function's output display list before the text
of the `li`.

Setup
-----

    >>> import wbemocks
    >>> _ = wbemocks.socket.patch().start()
    >>> _ = wbemocks.ssl.patch().start()
    >>> import browser

Set up the URL and web page, this is the content that we will be examining.

    >>> content = "<html><body><li>hello</li><li>world</li></body></html>"
    >>> url = browser.URL(wbemocks.socket.serve(content))
    >>> this_browser = browser.Browser()
    >>> this_browser.load(url)

Tests
-----

The layout tree should only contain two inline layout objects, corresponding to
  the body of the web page.
         
    >>> browser.print_tree(this_browser.document)
     DocumentLayout()
       BlockLayout(x=13, y=18, width=774, height=30.0, node=<html>)
         BlockLayout(x=13, y=18, width=774, height=30.0, node=<body>)
           BlockLayout(x=39, y=18, width=748, height=15.0, node=<li>)
           BlockLayout(x=39, y=33.0, width=748, height=15.0, node=<li>)

The text inside the `li` should be indented over by `2 * HSTEP`.
The bullet itself should be a black rectangle, 4 by 4 pixels, centered 
  vertically on the line and with a horizontal center one `HSTEP` from the 
  start of the line.
The `DrawRect` call should in the `paint` function's output display list before
  the text of the `li`.
  
    >>> wbemocks.print_list(this_browser.display_list)
    DrawRect(top=23.5 left=24 bottom=27.5 right=28 color=black)
    DrawText(top=20.25 left=39 bottom=32.25 text=hello font=Font size=12 weight=normal slant=roman style=None)
    DrawRect(top=38.5 left=24 bottom=42.5 right=28 color=black)
    DrawText(top=35.25 left=39 bottom=47.25 text=world font=Font size=12 weight=normal slant=roman style=None)

It is possible that multiple lines are contained in a `li`, and in these
  cases all lines should be indented and the bullet should be vertically 
  centered on the first line.

    >>> content = "<html><body><li>hello<br>world</li></body></html>"
    >>> url = browser.URL(wbemocks.socket.serve(content))
    >>> this_browser = browser.Browser()
    >>> this_browser.load(url)
    >>> wbemocks.print_list(this_browser.display_list)
    DrawRect(top=23.5 left=24 bottom=27.5 right=28 color=black)
    DrawText(top=20.25 left=39 bottom=32.25 text=hello font=Font size=12 weight=normal slant=roman style=None)
    DrawText(top=35.25 left=39 bottom=47.25 text=world font=Font size=12 weight=normal slant=roman style=None)

Verify that <li> tag widths are reduced to account for bullet spacing, not just their horizontal starting positions.

    >>> content = "<ul><li>Item1</li><li>Item2</li></ul>"
    >>> url = browser.URL(wbemocks.socket.serve(content))
    >>> this_browser = browser.Browser()
    >>> this_browser.load(url)
    >>> browser.print_tree(this_browser.document)
     DocumentLayout()
       BlockLayout(x=13, y=18, width=774, height=30.0, node=<html>)
         BlockLayout(x=13, y=18, width=774, height=30.0, node=<body>)
           BlockLayout(x=13, y=18, width=774, height=30.0, node=<ul>)
             BlockLayout(x=39, y=18, width=748, height=15.0, node=<li>)
             BlockLayout(x=39, y=33.0, width=748, height=15.0, node=<li>)
