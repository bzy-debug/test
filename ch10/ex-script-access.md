Tests for WBE Chapter 10 Exercise `Script access`
============================================

Implement the document.cookie JavaScript API. Reading this field
should return a string containing the cookie value ~~and parameters~~,
formatted similarly to the `Cookie` header. Writing to this field
updates the cookie value and parameters, just like receiving a
`Set-Cookie` header does. Also implement the `HttpOnly` cookie
parameter; cookies with this parameter cannot be read or written from
JavaScript.

Implementation is made easier since the browser supports one cookie
per host, so getting `document.cookie` will return either 0 or 1
cookie and setting it will overwrite the cookie (if allowable).

Tests
-----

Boilerplate.

    >>> import wbemocks
    >>> _ = wbemocks.socket.patch().start()
    >>> _ = wbemocks.ssl.patch().start()
    >>> wbemocks.NO_CACHE = True
    >>> import browser

Open an empty page to get the javascript instance

    >>> url = "http://wbemocks.wbemocks.chapter10-script-access/"
    >>> wbemocks.socket.respond_ok(url, "Empty")
    >>> this_browser = browser.Browser()
    >>> this_browser.new_tab(browser.URL(url))
    >>> js = this_browser.tabs[0].js

Check that javascript doesn't see any cookies, and that the cookie jar is empty.

    >>> js.run("console.log(document.cookie)")
    <BLANKLINE>
    >>> "wbemocks.wbemocks.chapter10-script-access" in browser.COOKIE_JAR
    False

Set the cookie and make sure the changes are applied.

    >>> js.run('void(document.cookie = "yoo=hoo")')
    >>> js.run("console.log(document.cookie)")
    yoo=hoo
    >>> browser.COOKIE_JAR["wbemocks.wbemocks.chapter10-script-access"]
    ('yoo=hoo', {})

Modify the key and see that change works.

    >>> js.run('void(document.cookie = "yoo=hey")')
    >>> js.run("console.log(document.cookie)")
    yoo=hey
    >>> browser.COOKIE_JAR["wbemocks.wbemocks.chapter10-script-access"]
    ('yoo=hey', {})

Set the cookie and use the parameter syntax.

    >>> js.run('void(document.cookie = "summer=blowout; SameSite=Lax")')
    >>> js.run("console.log(document.cookie)")
    summer=blowout
    >>> browser.COOKIE_JAR["wbemocks.wbemocks.chapter10-script-access"]
    ('summer=blowout', {'samesite': 'lax'})


Load a new page with a http only cookie.

    >>> url = "http://wbemocks.wbemocks.chapter10-script-access-http-only/"
    >>> wbemocks.socket.respond(url, b"HTTP/1.0 200 OK\r\nSet-Cookie: no=share;SameSite=None;HttpOnly\r\n\r\nEmpty")
    >>> this_browser = browser.Browser()
    >>> this_browser.new_tab(browser.URL(url))
    >>> js = this_browser.tabs[0].js

Reading from javascript should show nothing, but the cookie jar will have the
    cookie.

    >>> js.run("console.log(document.cookie)")
    <BLANKLINE>
    >>> browser.COOKIE_JAR["wbemocks.wbemocks.chapter10-script-access-http-only"][0]
    'no=share'

Assigning from javascript will not change the content of the cookie.

    >>> js.run('void(document.cookie = "overwrite=attempt")')
    >>> js.run("console.log(document.cookie)")
    <BLANKLINE>
    >>> browser.COOKIE_JAR["wbemocks.wbemocks.chapter10-script-access-http-only"][0]
    'no=share'
