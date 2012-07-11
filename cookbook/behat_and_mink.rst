Developing Web Applications with Behat and Mink
===============================================

You can use Behat to describe anything, that you can describe in business
logic. It's tools, gui applications, web applications. Most interesting part is
web applications. First, behavioral testing already exists in web world -
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
do things like this:

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
functional testing. Because both these tools are quite limited at some parts,
but succeed on others. For example, you can't use in-browser emulators for all
tests in your application, cuz this makes your tests become very slow. Also, you
can't do AJAX with headless browser.

You should use them both. But there comes a problem - these are very different
tools and they have much different API. Use both those API limits us very much
and in case of Behat, this problem become even worse, cuz now you have single:

.. code-block:: gherkin

    When I go to "/news.php"

And this step should be somehow executed through one or another browser
emulator.

Here comes Mink. Mink is a browser emulators abstraction layer. It hides
emulators differences behind single, consistent API.

Just some of the benefits:

1. Single, consistent API.
2. Almost zero-configuration.
3. Support for both, in-browser and headless browser emulators.

Installing Mink
---------------

Mink is a php 5.3 library that you'll use inside your test and feature suites.
Before you begin, ensure that you have at least PHP 5.3.1 installed.

Mink integration into Behat happens thanks to MinkExtension. Extension takes
care of all the configuration and initialization of the Mink, leaving only fun
parts to you.

Method #1 (Composer)
~~~~~~~~~~~~~~~~~~~~

The simplest way to install Behat with Mink is through Composer.

Create ``composer.json`` file in the project root:

.. code-block:: js

    {
        "require": {
            "behat/behat": "2.4.*@stable",
            "behat/mink": "1.4.*@stable",
            "behat/mink-extension": "*",
            "behat/mink-goutte-driver": "*",
            "behat/mink-selenium2-driver": "*"
        },
        "config": {
            "bin-dir": "bin/"
        }
    }

.. note::

    Note that we also installed two Mink drivers - goutte and
    selenium2. That's because by default, Composer installation
    of Mink doesn't include any driver - you should choose what
    to use by yourself.

    Easiest way to get started is to go with ``goutte`` and
    ``selenium2`` drivers, but note that there's bunch of other
    drivers available for Mink - read about them in Mink
    documentation.

Then download ``composer.phar`` and run ``install`` command:

.. code-block:: bash

    $ curl http://getcomposer.org/installer | php
    $ php composer.phar install

After that, you will be able to run Behat with:

.. code-block:: bash

    $ bin/behat -h

And this executable will already autoload all the needed classes
in order to **activate** MinkExtension through ``behat.yml``.

Now lets activate it:

.. code-block:: yaml

    # behat.yml
    default:
        extensions:
            Behat\MinkExtension\Extension:
                goutte: ~
                selenium2: ~

You could check that extension is properly loaded by calling:

.. code-block:: bash

    $ bin/behat -dl

It should show you all the predefined web steps as MinkExtension will
automatically use bundled ``MinkContext`` if no user-defined context class found.

Method #2 (PHAR)
~~~~~~~~~~~~~~~~

Also, you can use Behat, Mink and MinkExtension as PHAR packages.

Download Behat:

.. code-block:: bash

    $ wget https://github.com/downloads/Behat/Behat/behat.phar

Download Mink:

.. code-block:: bash

    $ wget https://github.com/downloads/Behat/Mink/mink.phar

Download MinkExtension:

.. code-block:: bash

    $ wget https://github.com/downloads/Behat/MinkExtension/mink_extension.phar

Put them all in the same folder.
After that, you will be able to run Behat with:

.. code-block:: bash

    $ php behat.phar -h

Now lets activate MinkExtension:

.. code-block:: yaml

    # behat.yml
    default:
        extensions:
            mink_extension.phar:
                mink_loader: mink.phar
                goutte: ~
                selenium2: ~

.. note::

    Behat extension name could be either of 3:

    1. Class name (if class is autoloaded) - best way in Composer installation
    2. PHAR file name
    3. Relative path to script, that will return new extension instance

You could check that extension is properly loaded by calling:

.. code-block:: bash

    $ php behat.phar -dl

It should show you all the predefined web steps as MinkExtension will
automatically use bundled ``MinkContext`` if no user-defined context class found.

``MinkContext`` for Behat requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

MinkExtension comes bundled with ``MinkContext``, which will be used automatically
by Behat as main context class if no user-defined context class found. That's why ``behat -dl``
shows you step definitions even when you haven't created custom ``FeatureContext`` class or
even ``features`` folder.

Writing your first Web Feature
------------------------------

Let's write a feature to test `Wikipedia <http://www.wikipedia.org/>`_ search abilities:

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

As you might see, urls in scenarios are relative, so we should provide correct
``base_url`` option for MinkExtension in our ``behat.yml``:

.. code-block:: yaml

    # behat.yml
    default:
        extensions:
            Behat\MinkExtension\Extension:
                base_url: http://en.wikipedia.org
                goutte: ~
                selenium2: ~

Now, run your feature (if installed through Composer):

.. code-block:: bash

    $ bin/behat features/search.feature

Or phar version:

.. code-block:: bash

    $ php behat.phar features/search.feature

You'll see output like this:

.. image:: /images/mink-wikipedia-2-scenarios.png
   :align: center

Test In-Browser - `selenium2` Session
-------------------------------------

Ok. We've successfully described wikipedia search and Behat tested it flawlessly. But
what about search field autocompletion? It's done using JS and AJAX, so we
can't use default headless session to test it - we need ``javascript`` session
and Selenium2 browser emulator for that task.

Selenium2 gives you ability to take full controll of real browser with clean
consistent proxy API. And Mink uses this API extensively in order to use same
Mink API and steps to do **real** actions in **real** browser.

All you need to do is install Selenium:

1. Download latest Selenium jar from the: http://seleniumhq.org/download/
2. Run Selenium2 jar before your test suites (you can start this proxy during system startup):

   .. code-block:: bash

        java -jar selenium-server-*.jar

That's it. Now you should create specific scenario in order it to be runnable
through Selenium:

.. code-block:: gherkin

    Scenario: Searching for a page with autocompletion
      Given I am on "/wiki/Main_Page"
      When I fill in "search" with "Behavior Driv"
      And I wait for the suggestion box to appear
      Then I should see "Behavior Driven Development"

Now, we need to tell Behat and Mink to run this scenario in different session
(with different browser emulator). Mink comes with special :doc:`hook </guides/3.hooks>`,
that searches ``@javascript`` or ``@mink:selenium2`` tag before scenario and switches
current Mink session to Selenium2 (in both cases). So, let's simply add this tag to
our scenario:

.. code-block:: gherkin

    @javascript
    Scenario: Searching for a page with autocompletion
      Given I am on "/wiki/Main_Page"
      When I fill in "search" with "Behavior Driv"
      And I wait for the suggestion box to appear
      Then I should see "Behavior-driven development"

Now run your feature again:

.. code-block:: bash

    $ bin/behat features/search.feature

And of course, you'll get:

.. image:: /images/mink-wikipedia-2.5-scenarios.png
   :align: center

That's because you have used custom ``Then I wait for the suggestion box to appear``
step, but not defined it yet. In order to do it, we will need to create our own
``FeatureContext`` class (at last).

Defining our own ``FeatureContext``
-----------------------------------

The easiest way to create context class is to ask Behat do it for you:

.. code-block:: bash

    $ bin/behat --init

This command will create ``features/bootstrap`` folder and
``features/bootstrap/FeatureContext.php`` class for you.

Now lets try to run our feature again (just to check that everything works):

.. code-block:: bash

    $ bin/behat features/search.feature

Oh... Now Behat tells us that all steps are undefined. What's happening there?

As we've created our own context class, MinkExtension stopped using own bundled
context class as main context and Behat uses your very own ``FeatureContext`` instead,
which of course doesn't have those Mink steps **yet**. Let's add them.

There's multiple ways to bring bundled with MinkExtension steps into your own
context class. Simplest one is to use inheritance. Just extend your context from
``Behat\MinkExtension\Context\MinkContext`` instead of base ``BehatContext``:

.. code-block:: php

    <?php

    use Behat\Behat\Context\ClosuredContextInterface,
        Behat\Behat\Context\TranslatedContextInterface,
        Behat\Behat\Context\BehatContext,
        Behat\Behat\Exception\PendingException;
    use Behat\Gherkin\Node\PyStringNode,
        Behat\Gherkin\Node\TableNode;

    use Behat\MinkExtension\Context\MinkContext;

    /**
     * Features context.
     */
    class FeatureContext extends MinkContext
    {
    }

To check that all ``MinkExtension`` steps are here again, run:

.. code-block:: bash

    $ bin/behat -dl

If all works properly, you should see something like this:

.. image:: /images/mink-definitions.png
   :align: center

Finally, lets add our custom ``wait`` step to context:

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
simple jQuery instruction.

Run feature again and:

.. image:: /images/mink-wikipedia-3-scenarios.png
   :align: center

Voila!

.. tip::

    Context isolation is very important thing in functional tests. But
    restarting the browser after each scenario could slow your feature suite
    very much. So, by default Mink tries hard to reset your browser session
    without reloading it (cleans all domain cookies).

    In some cases it might be not enough (when you use ``http-only`` cookies for
    example). In that case, just add ``@insulated`` tag to your scenario.
    Browser in this case will be fully reloaded and cleaned (before scenario):

    .. code-block:: gherkin

        Feature: Some feature with insulated scenario

          @javascript @insulated
          Scenario: isolated scenario
            #...

Going further
-------------

Read more cookbook articles on Behat and Mink interactions:

* :doc:`/cookbook/using_the_profiler_with_minkbundle`
* :doc:`/cookbook/intercepting_the_redirections`
