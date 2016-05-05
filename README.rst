Autologin: Automatic login for web spiders
==========================================

.. image:: https://img.shields.io/travis/TeamHG-Memex/autologin/master.svg
   :target: http://travis-ci.org/TeamHG-Memex/autologin
   :alt: Build Status

.. image:: https://codecov.io/github/TeamHG-Memex/autologin/coverage.svg?branch=master
   :target: https://codecov.io/github/TeamHG-Memex/autologin?branch=master
   :alt: Code Coverage


AutoLogin is a library that makes it easier for web spiders to
**crawl websites that require login**.
Provide it with credentials and a URL or the html source of a page
(normally the homepage), and it will attempt to login for you.
Cookies are returned to be used by your spider.

The goal of Autologin is to make it easier for web spiders to crawl websites
that require authentication
**without having to re-write login code for each website.**

Autologin can be used as a library, on the command line, or as a service.
You can make use of Autologin without generating http requests,
so you can drop it right into your spider without worrying about
impacting rate limits.

Autologin works on Python 2.7 and 3.3+.

.. contents::

Features
--------

* Automatically find login forms and fields
* Obtain authenticated cookies
* Obtain form requests to submit from your own spider
* Extract links to login pages
* Use as a library with or without making http requests
* Command line client
* Web service
* UI for managing login credentials


Quickstart
----------

Don't like reading documentation?

::

    from autologin import AutoLogin

    url = 'https://reddit.com'
    username = 'foo'
    password = 'bar'
    al = AutoLogin()
    cookies = al.auth_cookies_from_url(url, username, password)

You now have a `cookiejar <https://docs.python.org/2/library/cookielib.html>`_
that you can use in your spider.  Don't want a cookiejar?

::

    cookies.__dict__

You now have a dictionary.


Installation
------------

This is not (yet) registered on PyPi::

    $ pip install git+https://github.com/TeamHG-Memex/autologin.git

Autologin depends on
`Formasaurus <https://github.com/TeamHG-Memex/Formasaurus>`_
for field and form classification, which has quite a lot of dependencies.
These packages may require extra steps to install, so the command above
may fail.
In this case install dependencies manually, one by one
(follow their install instructions).

A recent ``pip`` is recommended (update it with ``pip install pip -U``).
On Ubuntu, the following packages are required (use ``python3-dev`` if you
are using Python 3)::

    $ apt-get install build-essential libssl-dev libffi-dev python-dev


Auth cookies from URL
---------------------

This method makes an http request to the URL,
extracts the login form (if there is one),
fills the fields and submits the form.
It then return any cookies it has picked up::

    cookies = al.auth_cookies_from_url(url, username, password)

Note that it returns all cookies, they may be session cookies rather
than authenticated cookies.

This call is blocking, and uses Crhochet to run the Twisted reactor
and a Scrapy spider in a separate thread.
If you have a Scrapy spider (or use Twisted in some other way),
use the HTTP API, or the non-blocking API (it's not documented,
see ``http_api.AutologinAPI._login``).

There are two opitonal arguments for ``AutoLogin.auth_cookies_from_url``:

- ``splash_url`` if set, `Splash <http://splash.readthedocs.org>`_
  will be used to make all requests. Use it if your cawler also uses
  splash and the session is tied to IP and User-Agent, or for Tor sites.
- ``settings`` is a dictionary with Scrapy settings to override.
  Use it e.g. to set a custom User-Agent with scrapy ``USER_AGENT`` option.

An example of using this options::

    cookies = al.auth_cookies_from_url(
        url, username, password,
        splash_url='http://127.0.0.1:8050',
        settings={'USER_AGENT': 'Mozilla/2.02 [fr] (WinNT; I)'})


Login request
-------------

This method extracts the login form (if there is one),
fills the fields and returns a dictionary with the form url and args
for your spider to submit. No http requests are made::

    >>> al.login_request(html_source, username, password, base_url=None)
    {'body': 'login=admin&password=secret',
     'headers': {b'Content-Type': b'application/x-www-form-urlencoded'},
     'method': 'POST',
     'url': '/login'}

Relative form action will be resolved against the ``base_url``.


Command Line
------------

::

    $ autologin
    usage: autologin [-h] [--splash-url SPLASH_URL] [--show-in-browser]
                     username password url


HTTP API
--------

You can start the autologin HTTP API with::

    $ autologin-http-api

and use ``/login-cookies`` endpoint. Make a POST request with JSON body.
The following arguments are supported:

- ``url`` (required): url of the site where we would like to login
- ``username`` (optional): if not provided, it will be fetched from the
  login keychain
- ``password`` (optional): same as ``username``
- ``splash_url`` (optional): if set, `Splash <http://splash.readthedocs.org>`_
  will be used to make all requests. Use it if your cawler also uses
  splash and the session is tied to IP and User-Agent, or for Tor sites.
- ``settings`` (optional) - a dictionary with Scrapy settings to override.
  Use it e.g. to set a custom User-Agent with scrapy ``USER_AGENT`` option.

If ``username`` and ``password`` are not provided, autologin tries to find
them in the login keychain. If no matching credentials are found (they are
matched by domain, not by precise url), then human is expected to eventually
provide them in the keychain UI, or mark domain as "skipped".

Response is JSON with a ``status`` field with the following possible values:

- ``error`` status means an error occured, ``error`` field has more info
- ``skipped`` means that domain is maked as "skipped" in keychain UI
- ``pending`` means there is an item in keychain UI (or it was just created),
  and no credentials have been entered yet
- ``solved`` means that cookies were obtained, they are returned in the
  ``cookies`` field, in ``Cookie.__dict__`` format.


Keychain UI
-----------

Start keychain UI with::

    $ autologin-server

Note that both ``autologin-server`` and ``autologin-http-api``
are not protected by any authentication.


Contributors
------------

Source code and bug tracker are on github:
https://github.com/TeamHG-Memex/autologin.

Run tests with ``tox``::

    $ tox

Splash support is not tested directly here, but there are indirect tests for it
in the `undercrawler <https://github.com/TeamHG-Memex/undercrawler>`_
test suite.


License
-------

License is MIT.