Developing Web Applications with Behat and Mink
===============================================

You can use Behat to describe anything, that you can describe in business
logic. It's tools, gui applications, web applications. Most interesting part is
web applications. First, behavioral testing is already exists in web world -
it's called *functional* or *acceptance* testing. Almost all popular
frameworks and languages provide functional testing tools. Today we'll talk
about how to use Behat for functional testing of web applications.

Understanding Mink
------------------

One of the most important parts in the web is a browser. Browser is the window,
through which web application users interact with application and other users.
User is always talking with web application through browser application.

So, in order to test web application, we should transform user actions into
steps and expectations - with Behat it's quite simple already. And next part
is much harder - do this actions and test expectations. How to programmatically
do think like this:

.. code-block:: gherkin

    Given I am on "/index.php"

You'll need something to simulate browser application. Scenario steps would
simulate user and browser emulator would simulate browser with wich user
interacts in order to talk with web application.

Now the real problem. We have 2 completely different type of solutions:

* *Headless browser emulators* - browser emulators, that can be executed fully
  without GUI through console. Such emulators can do HTTP request and emulate
  browser applications on high level (HTTP stack), but on lower level (JS, CSS)
  they are totally limited. But they are much faster than real browsers, cuz
  you don't need to parse CSS or execute JS in order to open pages or click
  links with them.

* *In-browser emulators* - this emulators works with real browsers, taking
  full controll of them and using them as zombies for their testing needs. This
  way, you'll have standart fully-configured real browser, which you will be
  able to controll. CSS styling, JS and AJAX execution - all supported out of
  the box.

The problem is we need both these emulator type in order to do successful
functional testing. Because both this tools is quite limited at some parts, but
succeed on others. For example, you can't use in-browser emulators for all
tests in your application, cuz this your tests become very slow. Also, you
can't do AJAX with headless browser.

You should use them both. But there comes a problem - this is very different
tools and they have much different API. Use both those API limits us very much
and in case of Behat, this problem become even worse, cuz now you have single:

.. code-block:: gherkin

    When I go to "/news.php"

And this step should be somehow executed through one or another browser
emulator.

Here comes the Mink. Mink is a browser emulators abstraction layer. It hides
emulators differences behind single, consistent API.

Just some of the benefits:

1. Single, consistent API.
2. Almost zero-configuration.
3. Support for both in-browser and headless browser emulators.

Installink Mink
---------------

Mink is php 5.3 library that you'll use inside your test and feature suites.
Before you begin, ensure that you have at least PHP 5.3.1 installed.

Method #1 (PEAR)
~~~~~~~~~~~~~~~~

The simplest way to install Behat is through PEAR:

.. code-block:: bash

    $ pear channel-discover pear.behat.org
    $ pear install behat/mink-beta

Now, you can use Mink in your projects simply by including it:

.. code-block:: php

    require_once 'mink/autoload.php';

Method #2 (PHAR)
~~~~~~~~~~~~~~~~

Also, you can use mink phar package:

.. code-block:: bash

    $ wget https://github.com/downloads/Behat/Mink/mink-1.0.0RC3.phar

Now you can require phar package in your project:

.. code-block:: php

    require_once 'mink-1.0.0RC3.phar';

``MinkContext`` for Behat requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mink comes with ready to work Behat ``FeatureContext`` implementation. It's
called ``MinkContext`` and it uses PHPUnit assertions internally, so you need
to install latest PHPUnit in order to use bundled with Mink steps:

.. code-block:: bash

    $ pear channel-discover pear.phpunit.de
    $ pear channel-discover components.ez.no
    $ pear channel-discover pear.symfony-project.com
    $ pear install phpunit/PHPUnit

Using ``MinkContext`` in Behat
------------------------------

Mink comes with ``Behat\Mink\Behat\Context\MinkContext`` context class. You
could either inherit or subcontext it (see :doc:`FeatureContext </guides/4.context>`
chapter):

.. code-block:: php

    <?php # features/bootstrap/FeatureContext.php

    use Behat\Behat\Context\ClosuredContextInterface,
        Behat\Behat\Context\BehatContext,
        Behat\Behat\Exception\Pending;

    use Behat\Gherkin\Node\PyStringNode,
        Behat\Gherkin\Node\TableNode;

    require_once 'mink/autoload.php';
    // or, if you want to use phar from current dir:
    // require_once __DIR__ . '/mink-1.0.0RC3.phar';

    class FeatureContext extends Behat\Mink\Behat\Context\MinkContext
    {
    }

Notice, that we've extended ``Behat\Mink\Behat\Context\MinkContext`` context
class. This context comes bundled with bunch of very useful *web* steps. To
see all available steps and to check, that Mink is included correctly - call:

.. code-block:: bash

    $ behat --definitions

If all works properly, you should see something like this:

.. image:: /images/mink-definitions.png
   :align: center

Now, all navigation steps in Mink uses relative paths. In order to be able to
make full URL's from relative paths, Mink should know about ``base_url`` value.
And you can help Mink with it:

.. code-block:: yaml

    # behat.yml
    default:
      context:
        parameters:
          base_url: http://en.wikipedia.org/

.. tip::

    ``base_url`` should always end up with ``/``.

Writing your first Web Feature
------------------------------

Let's for example write a feature to test `Wikipedia <http://www.wikipedia.org/>`_
search abilities:

.. code-block:: gherkin

    # features/search.feature
    Feature: Search
      In order to see a word definition
      As a website user
      I need to be able to search for a word

      Scenario: Searching for a page that does exist
        Given I am on "/wiki/Main_Page"
        When I fill in "search" with "Behavior Driven Development"
        And I press "searchButton"
        Then I should see "agile software development"

      Scenario: Searching for a page that does NOT exist
        Given I am on "/wiki/Main_Page"
        When I fill in "search" with "Glory Driven Development"
        And I press "searchButton"
        Then I should see "Search results"

We have two scenarios here:

* *Searching for a page that does exist* - describes, how Wikipedia searches
  for pages, that does exist in Wikipedia index.

* *Searching for a page that does NOT exist* - describes, how Wikipedia
  searches for pages, that does not exist in Wikipedia index.

Now, run your feature:

.. code-block:: bash

    $ behat features/search.feature

You'll see output like this:

.. image:: /images/mink-wikipedia-2-scenarios.png
   :align: center

Test In-Browser - Sahi Session
------------------------------

Ok. We've successfully tested wikipedia search and it works flawlessly. But
what about search field autocompletion? It's done using JS and AJAX, so we
can't use default headless session to test it - we need ``javascript`` session
and Sahi browser emulator for that task.

Sahi gives you ability to take full controll of real browser with clean
consistent proxy API. And Mink uses this API extensively in order to use same
Mink API and steps to do **real** actions in **real** browser.

All you need to do is install Sahi:

1. Download and run the Sahi jar from the http://sahi.co.in/w/

2. Run sahi proxy before your test suites (you can start this proxy during system startup):

   .. code-block:: bash

        cd $YOUR_PATH_TO_SAHI/bin
        ./sahi.sh

That's it. Now you should create specific scenario in order it to be runnable
through Sahi:

.. code-block:: gherkin

    Scenario: Searching for a page with autocompletion
      Given I am on "/wiki/Main_Page"
      When I fill in "search" with "Behavior Driv"
      And I wait for the suggestion box to appear
      Then I should see "Behavior Driven Development"

Now, we need to tell Behat and Mink to run this scenario in different session (
with different browser emulator). Mink comes with special :doc:`hook </guides/3.hooks>`,
that searches ``@javascript`` or ``@mink:sahi`` tag before scenario and switches
current Mink session to Sahi (in both cases). So, let's simply add this tag to
our scenario:

.. code-block:: gherkin

    @javascript
    Scenario: Searching for a page with autocompletion
      Given I am on "/wiki/Main_Page"
      When I fill in "search" with "Behavior Driv"
      And I wait for the suggestion box to appear
      Then I should see "Behavior Driven Development"

Now run your feature again:

.. code-block:: bash

    $ behat features/search.feature

And of course, you'll get:

.. image:: /images/mink-wikipedia-2.5-scenarios.png
   :align: center

It's because you indeed doesn't have ``Then I wait for the suggestion box to appear``
step. But don't worry, Behat already provided the regex and function snippets
for use and Mink makes new steps writing a breathe:

.. code-block:: php

    /**
     * @Then /^I wait for the suggestion box to appear$/
     */
    public function iWaitForTheSuggestionBoxToAppear()
    {
        $this->getSession()->wait(5000,
            "$('.suggestions-results').children().length > 0"
        );
    }

That simple. We get current session and send JS command to wait (sleep) for 5
seconds or until expression in second argument returns true. Second argument is
simple jQuery instruction. Run feature again and:

.. image:: /images/mink-wikipedia-3-scenarios.png
   :align: center

Voila!
