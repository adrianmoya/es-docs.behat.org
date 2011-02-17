Gherkin DSL
===========

Gherkin is the language that Behat understands. It is a `Business Readable, Domain Specific Language <http://martinfowler.com/bliki/BusinessReadableDSL.html>`_ that lets you describe software’s behaviour without detailing how that behaviour is implemented.

Gherkin serves two purposes – documentation and automated tests. The third is a bonus feature – when it yells in red it’s talking to you, telling you what code you should write.

Gherkin’s parser uses Symfony Translation component and xliff translations, so grammar exists in different flavours for many spoken languages (42 at the time of writing), so that your team can use the keywords in your own language.

To check if Behat & Gherkin supports your, let's say french, language, run:

.. code-block:: bash

    behat --usage --lang=fr

There are a few conventions.

* Single Gherkin source file contains a description of a single feature.
* Source files have ``.feature`` extension.

Gherkin Syntax
--------------

Like Python and YAML, Gherkin is a line-oriented language that uses indentation to define structure. Line endings terminate statements (eg, steps). Either spaces or tabs may be used for indentation (but spaces are more portable). Most lines start with a keyword.

Parser divides the input into features, scenarios and steps. When you run the feature the trailing portion (after the keyword) of each step is matched to a PHP callback.

A Gherkin source file usually looks like this:

.. code-block:: gherkin

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

First line starts the feature. Lines 2-4 are unparsed text, which is expected to describe the business value of this feature. Line 6 starts a scenario. Lines 7-13 are the steps for the scenario. Line 15 starts next scenario and so on.
