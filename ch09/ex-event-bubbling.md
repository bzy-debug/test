Tests for WBE Chapter 9 Exercise `Event Bubbling`
============================================

Right now, you can attach a click handler to `a` elements, but not to
anything else. Fix this. One challenge you’ll face is that when you
click on an element, you also click on all its ancestors. On the web,
this sort of quirk is handled by event bubbling: when an event is
generated on an element, listeners are run not just on that element
but also on its ancestors. Implement event bubbling, and make sure
listeners can call stopPropagation on the event object to stop
bubbling the event up the tree. Double-check that clicking on links
still works, and make sure `preventDefault` still successfully prevents
clicks on a link from actually following the link.

Do not change the return value of `dispatchEvent` (this is a
publically accessible function that web pages can call).

Tests
-----

Boilerplate.

    >>> import wbemocks
    >>> _ = wbemocks.socket.patch().start()
    >>> _ = wbemocks.ssl.patch().start()
    >>> import browser

Set up the webpage and script links.

    >>> script = """
    ... document.querySelectorAll('div')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('div saw a click');
    ...   });
    ... document.querySelectorAll('form')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('form saw a click');
    ...   });
    ... document.querySelectorAll('input')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('input saw a click');
    ...   });
    ... """
    >>> js_url = wbemocks.socket.serve(script)
    >>> body = "<script src=" + str(js_url) + "></script>"
    >>> body += "<div><form><input name=bubbles value=sugar></form></div>"
    >>> html_url = wbemocks.socket.serve(body)

Attach an event listener to each nested element.
Click an show that all event listeners are called in the correct order.

    >>> this_browser = browser.Browser()
    >>> this_browser.new_tab(browser.URL(html_url))
    >>> this_browser.handle_click(wbemocks.ClickEvent(20, 24 + this_browser.chrome.bottom))
    input saw a click
    form saw a click
    div saw a click
    >>> js = this_browser.active_tab.js
    >>> js.run("document.querySelectorAll('input')[0].getAttribute('value')")
    ''

Setup a new webpage with the same content but a different script.
This time prevent the default in the input.

    >>> script = """
    ... document.querySelectorAll('div')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('div saw a click');
    ...   });
    ... document.querySelectorAll('form')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('form saw a click');
    ...   });
    ... document.querySelectorAll('input')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('input saw a click');
    ...     e.preventDefault();
    ...   });
    ... """
    >>> js_url = wbemocks.socket.serve(script)
    >>> body = "<script src=" + str(js_url) + "></script>"
    >>> body += "<div><form><input name=bubbles value=sugar></form></div>"
    >>> html_url = wbemocks.socket.serve(body)

    >>> this_browser = browser.Browser()
    >>> this_browser.new_tab(browser.URL(html_url))
    >>> this_browser.handle_click(wbemocks.ClickEvent(20, 24 + this_browser.chrome.bottom))
    input saw a click
    form saw a click
    div saw a click
    >>> js = this_browser.active_tab.js
    >>> js.run("document.querySelectorAll('input')[0].getAttribute('value')")
    'sugar'

Stopping propagation should also work.

    >>> script = """
    ... document.querySelectorAll('div')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('div saw a click');
    ...   });
    ... document.querySelectorAll('form')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('form saw a click');
    ...     e.stopPropagation();
    ...   });
    ... document.querySelectorAll('input')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('input saw a click');
    ...   });
    ... """
    >>> js_url = wbemocks.socket.serve(script)
    >>> body = "<script src=" + str(js_url) + "></script>"
    >>> body += "<div><form><input name=bubbles value=sugar></form></div>"
    >>> html_url = wbemocks.socket.serve(body)

    >>> this_browser = browser.Browser()
    >>> this_browser.new_tab(browser.URL(html_url))
    >>> this_browser.handle_click(wbemocks.ClickEvent(20, 24 + this_browser.chrome.bottom))
    input saw a click
    form saw a click
    >>> js = this_browser.active_tab.js
    >>> js.run("document.querySelectorAll('input')[0].getAttribute('value')")
    ''

Both should be able to work on the same click.

    >>> script = """
    ... document.querySelectorAll('div')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('div saw a click');
    ...   });
    ... document.querySelectorAll('form')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('form saw a click');
    ...   });
    ... document.querySelectorAll('input')[0].addEventListener('click',
    ...   function(e) {
    ...     console.log('input saw a click');
    ...     e.preventDefault();
    ...     e.stopPropagation();
    ...   });
    ... """
    >>> js_url = wbemocks.socket.serve(script)
    >>> body = "<script src=" + str(js_url) + "></script>"
    >>> body += "<div><form><input name=bubbles value=sugar></form></div>"
    >>> html_url = wbemocks.socket.serve(body)

    >>> this_browser = browser.Browser()
    >>> this_browser.new_tab(browser.URL(html_url))
    >>> this_browser.handle_click(wbemocks.ClickEvent(20, 24 + this_browser.chrome.bottom))
    input saw a click
    >>> js = this_browser.active_tab.js
    >>> js.run("document.querySelectorAll('input')[0].getAttribute('value')")
    'sugar'






Test: Form submit still works after event bubbling changes

    >>> wbemocks.socket.respond_ok("http://example.com/submit", "")
    >>> body = "<form action='http://example.com/submit' method='post'><button>Click</button></form>"
    >>> html_url = wbemocks.socket.serve(body)

    >>> this_browser = browser.Browser()
    >>> this_browser.new_tab(browser.URL(html_url))
    >>> this_browser.handle_click(wbemocks.ClickEvent(18, 25 + this_browser.chrome.bottom))
    >>> this_browser.active_tab.url
    URL(scheme=http, host=example.com, port=80, path='/submit')
