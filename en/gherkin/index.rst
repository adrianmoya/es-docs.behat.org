The Gherkin Language
====================

Behat revolves around the idea of writing human-readable sentences that can
be converted into real tests against your application. The format used to
write these sentences is called *Gherkin*, and is a "`Business Readable, Domain Specific Language`_"
that lets you describe your software’s behavior without detailing how that
behavior is implemented.

Gherkin serves two purposes – documentation and automated tests. A third,
bonus feature, is that when it yells at you in red, it’s talking to you,
using real, human language to tell you what code you should write.

If you're still new to Behat, jump into the :doc:`/quick_intro` first and
then come back here to learn more about Gherkin.

Gherkin Syntax
--------------

Like YAML or Python, Gherkin is a line-oriented language that uses indentation
to define structure. Line endings terminate statements (e.g., steps) and either
spaces or tabs may be used for indentation (but spaces are more portable).
Finally, most lines start with a keyword:

.. code-block:: gherkin
    :linenos:

    Feature: Some terse yet descriptive text of what is desired
      In order to realize a named business value
      As an explicit system actor
      I want to gain some beneficial outcome which furthers the goal
    
      Scenario: Some determinable business situation
        Given some precondition
          And some other precondition
         When some action by the actor
          And some other action
          And yet another action
         Then some testable outcome is achieved
          And something else we can check happens too
    
      Scenario: A different situation
          ...

The parser divides the input into features, scenarios and steps. When you
run the execute the feature, the trailing portion of each step (the part
after the keywords like ``Given``, ``And``, ``When``, etc) is matched to
a regular expression, which executes a PHP callback function.

Let's walk through the above example:

* lines 2-4 are unparsed text, which describes the business value of the feature
* line 6 starts the scenario, and contains a description of the scenario
* lines 7-13 are the steps for the scenario, which are matched to regular
  expressions defined elsewhere
* line 15 starts the next scenario, and so on

Gherkin in Many Languages
-------------------------

Gherkin is available in many languages, allowing you to write your stories
in your language, using keywords from your language. In other words, if you
speak French, you can use the world ``Fonctionnalité`` instead of ``Feature``.

To check if Behat & Gherkin supports your language (for example, French), run:

.. code-block:: bash

    behat --usage --lang=fr

.. _`Business Readable, Domain Specific Language`: http://martinfowler.com/bliki/BusinessReadableDSL.html