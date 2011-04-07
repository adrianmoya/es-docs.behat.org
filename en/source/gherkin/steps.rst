Steps & Step Definitions
========================

:doc:`features` consist of steps, also known as Givens, Whens and Thens.

Behat doesn’t technically distinguish between these three kind of steps. However, we strongly recommend that you do! These words have been carefully selected for their purpose, and you should know what the purpose is to get into the BDD mindset.

Robert C. Martin has written a `great post <http://blog.objectmentor.com/articles/2008/11/27/the-truth-about-bdd>`_ about BDD’s Given-When-Then concept where he thinks of them as a finite state machine.

Steps
-----

Given
~~~~~

The purpose of givens is to **put the system in a known state** before the user (or external system) starts interacting with the system (in the When steps). Avoid talking about user interaction in givens. If you had worked with usecases, you would call this preconditions.

Examples:

* Create records (model instances) / set up the database state.
* It’s ok to call into the layer “inside” the UI layer here (in symfony: talk to the models).
* Log in a user (An exception to the no-interaction recommendation. Things that “happened earlier” are ok).

And for all the symfony users out there – we recommend using a Given with a multiline table argument set up records instead of fixtures. This way you can read the scenario and make sense out of it without having to look elsewhere (at the fixtures).

When
~~~~

The purpose of When steps is to **describe the key action** the user performs (or, using Robert C. Martin’s metaphor, the state transition).

Examples:

* Interact with a web page (BehatBundle implements web steps, that should mostly go into When steps).
* Interact with some other user interface element.
* Developing a library? Kicking off some kind of action that has an observable effect somewhere else.

Then
~~~~

The purpose of Then steps is to **observe outcomes**. The observations should be related to the business value/benefit in your feature description. The observations should also be on some kind of output – that is something that comes out of the system (report, user interface, message) and not something that is deeply buried inside it (that has no business value).

Examples:

* Verify that something related to the Given+When is (or is not) in the output
* Check that some external system has received the expected message (was an email with specific content sent?)

.. note::

    While it might be tempting to implement Then steps to just look in the database – resist the temptation. You should only verify outcome that is observable for the user (or external system) and databases usually are not.

And, But
~~~~~~~~

If you have several givens, whens or thens you can write:

.. code-block:: gherkin

    Scenario: Multiple Givens
      Given one thing
      Given an other thing
      Given yet an other thing
      When I open my eyes
      Then I see something
      Then I don't see something else

Or you can make it read more fluently by writing:

.. code-block:: gherkin

    Scenario: Multiple Givens
      Given one thing
        And an other thing
        And yet an other thing
       When I open my eyes
       Then I see something
        But I don't see something else

To Behat steps beginning with And or But are exactly the same kind of steps as all the others.

Step Definitions
----------------

Step definitions are defined in PHP files under ``features/steps/*.php``. Here is a simple example:

.. code-block:: php

    <?php
    // features/steps/elephants_steps.php

    $steps->Given('/^I have (\d+) elephants in my belly$/', function($world, $count) {
        // Some PHP code here
    });

A step definition is a simple callback tied to custom regex. Step definitions can take 1 or more arguments, identified by groups in the Regexp (and an equal number of arguments to the callback plus 1). "plus 1" is a ``$world`` :doc:`../behat/environment` object argument. :doc:`../behat/environment` is a container object shared across a scenario (and the associated background) steps.

Then there are Steps. Steps are declared in your ``features/*.feature`` files. Here is an example:

.. code-block:: gherkin

    Given I have 93 elephants in my belly

A step is analogous to a callback invocation. In this example, you’re “calling” the step definition above with one argument – the string “93”. Behat matches the Step against the Step Definition’s Regexp and takes all of the matches from that match and passes them to the callback (after ``$world``, ofcourse).

Step Definitions start with an `adjective <http://www.merriam-webster.com/dictionary/given>`_ or an `adverb <http://www.merriam-webster.com/dictionary/when>`_, and can be expressed in any of Behat’s supported languages. All Step definitions are loaded (and defined) before Behat starts to execute the features plain text.

When Behat executes the plain text, it will for each step look for a registered Step Definition with a matching Regexp. If it finds one it will execute its callback, passing all groups from the Regexp match as arguments to the callback (after ``$world``, ofcourse).

The adjective/adverb has **no** significance when Behat is registering or looking for Step Definitions.

Successful
~~~~~~~~~~

When Behat finds a matching Step Definition it will execute it. If the block in the step definition doesn’t raise an Exception, the step is marked as passed (green). What you return from a Step Definition has no significance what so ever. Only Exceptions or their absence matters.

Undefined
~~~~~~~~~

When Behat can’t find a matching Step Definition the step gets marked as yellow, and all subsequent steps in the scenario are skipped. If your test suite has undefined steps – Behat will still exit with ``0`` code. But if you'll specify ``--strict`` option, then Behat will exit with ``1`` (fail).

Pending
~~~~~~~

When a Step Definition’s callback throws the ``Behat\Behat\Exception\Pending``, the step is marked as yellow (as with undefined ones), reminding you that you have work to do. If your test suite has undefined steps – Behat will still exit with ``0`` code. But if you'll specify ``--strict`` option, then Behat will exit with ``1`` (fail).

Failed
~~~~~~

When a Step Definition’s callback is executed and raises an Exception, the step is marked as red. What you return from a Step Definition has no significance what so ever. Returning null or false will not cause a step definition to fail. If suite has failed steps - Behat will return ``1`` to console.

Skipped
~~~~~~~

Steps that follow undefined, pending or failed steps are never executed (even if there is a matching Step Definition), and are marked cyan.

Ambiguous
~~~~~~~~~

Consider these step definitions:

.. code-block:: php

    <?php
    // features/steps/amb_steps.php

    $steps->Given('/Three (.*) mice/', function($world, $disability) {
    });

    $steps->Given('/Three blind (.*)/', function($world, $animal) {
    });

And a plain text step:

.. code-block:: gherkin

    Given Three blind mice

Behat can’t make a decision about what Step Definition to execute, and wil raise a ``Behat\Behat\Exception\Ambiguous`` exception telling you to fix the ambiguity.

Redundant
~~~~~~~~~

In Behat you’re not allowed to use a regexp more than once in a Step Definition (even across files, even with different code inside the Proc), so the following would cause a ``Behat\Behat\Exception\Redundant`` error:

.. code-block:: php

    <?php
    // features/steps/redundant_steps.php

    $steps->Given('/Three (.*) mice/', function($world, $disability) {
        // some code
    });

    $steps->Given('/Three (.*) mice/', function($world, $disability) {
        // some other code
    });

Step Definition Localization
----------------------------

Sometimes, you might need to provide multiple languages of your step definitions. For example, when you're writing some assertion library with bundled Behat steps. In this case, Behat supports definition translations. To translate your definitions, create ``features/steps/i18n`` folder and place regex translastions there in `XLIFF <http://en.wikipedia.org/wiki/XLIFF>`_ format:

.. code-block:: xml

    <!-- features/steps/i18n/ru.xliff --->
    <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
      <file original="global" source-language="en" target-language="ru" datatype="plaintext">
        <header />
        <body>
          <trans-unit id="i-have-entered">
            <source>/^I have entered (\d+) into calculator$/</source>
            <target>/^Я набрал число (\d+) на калькуляторе$/</target>
          </trans-unit>
          <trans-unit id="i-have-clicked-plus">
            <source>/^I have clicked "+"$/</source>
            <target>/^Я нажал "+"$/</target>
          </trans-unit>
          <trans-unit id="i-should-see">
            <source>/^I should see (\d+) on the screen$/</source>
            <target>/^Я должен увидеть на экране (\d+)$/</target>
          </trans-unit>
        </body>
      </file>
    </xliff>

This translations will be used to match your localized features. For example, this is russian translation & it will be used in all features, written in ``# language: ru``. Also, Behat's ``--steps`` option accepts additional ``--lang`` option in which you can specify language to see available steps:

.. code-block:: bash

    behat --steps --lang=ru

will print all available steps in russian (if has translations).

Named step regexps
------------------

In some cases, you might need to use named regex arguments instead of default positioned ones. Behat allows this from version 1.1:

.. code-block:: php

    <?php
    
    $steps->Given('/^(?P<who>\w+) did (?P<what>\w+)$/', function($world, $who, $what) {});

In this case arguments will be mapped accordingly to their matcher names. This means, that the privous example is equivalent to the following:

.. code-block:: php

    <?php
    
    $steps->Given('/^(?P<what>\w+) was done by (?P<who>\w+)$/', function($world, $who, $what) {});

.. note::

    Don't mix named arguments with non-named ones. This would lead to unpredictable results!

Steps Organization
------------------

How do you name step definition files? What to put in each step definition? What not to put in step definitions? Here are some guidelines that will lead to better scenarios.

Grouping
~~~~~~~~

Technically it doesn’t matter how you name your step definition files and what step definitions you put in what file. You *could* have one giant file called allSteps.php and put all your step definitions there. That would be messy.

We recommend creating a ``*_steps.php`` file for each domain concept. For example, a good rule of thumb is to have one file for each major model/database table. In a Curriculum Vitae application we might have:

* features/steps/employee_steps.php
* features/steps/education_steps.php
* features/steps/experience_steps.php
* features/steps/authentication_steps.php

The three first ones would define all the ``Given``, ``When``, ``Then`` step definitions related to creating, reading, updating and deleting the various models. The last one defines step definitions related to logging in and out.

Step state
~~~~~~~~~~

It’s possible to keep object state in ``$world->variables`` inside your step definitions. Be careful about this as it might make your steps more tightly coupled and harder to reuse. There is no absolute rule here – sometimes it’s ok to use ``$world->variables``.
