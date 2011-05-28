Tags
====

Tags is a great way to organise your features and scenarios. Consider this example:

.. code-block:: gherkin

    @billing
    Feature: Verify billing

      @important
      Scenario: Missing product description

      Scenario: Several products

A Scenario or feature can have as many tags as you like. Just separate them with spaces:

.. code-block:: gherkin

    @billing @bicker @annoy
    Feature: Verify billing

Tags Inheritance
----------------

If a tag exists on a ``Feature``, Behat will think, that it exists on it's ``Scenario`` or ``Scenario Outline`` too.

Running a Subset of Scenarios
-----------------------------

You can use the ``--tags`` option to tell Behat that you only want to run features or scenarios that have (or don’t have) certain tags. Examples:

.. code-block:: bash

    behat --tags @billing            # Runs both scenarios
    behat --tags @important          # Runs the first scenario
    behat --tags ~@important         # Runs the second scenario (Scenarios without @important)

    behat --tags @billing&&@important    # Runs the first scenario (Scenarios with @important AND @billing)
    behat --tags @billing,@important     # Runs both scenarios (Scenarios with @important OR @billing)

(Another way to “filter” what you want to run is to use the ``/path/to/file.feature:line`` (it will run scenario on specific line) pattern or the ``--name`` option).

Tags is also a great way to "link" your Behat features to other documents. For example, if you have to deal with old school requirements in a different system (Word, Excel, a wiki) you can refer to numbers:

.. code-block:: gherkin

    @BJ-x98.77 @BJ-z12.33
    Feature: Convert transaction

Another creative way to use tags is to keep track of where in the development process a certain feature is:

.. code-block:: gherkin

    @qa_ready
    Feature: Index projects

Tags are also used in :doc:`../behat/hooks`, which let you use tags to define what before and after code get run for what scenarios.

Logically ANDing and ORing Tags
-------------------------------

As you may have seen in the previous examples Behat allows you to use logical ANDs and ORs to help gain greater control of what features to run.

Tags which are comma separated are ORed (run scenarios which match @important OR @billing):

.. code-block:: bash

    behat --tags @billing,@important

Tags which are ``&&`` separates are ANDed (run scenarios which match @important AND @billing):

.. code-block:: bash

    behat --tags @billing&&@important

You can combine these two methods to create powerful selection criteria (run scenarios which match: (@billing OR @WIP) AND @important):

.. code-block:: bash

    behat --tags @billing,@wip&&@important

Skipping both @todo and @wip tags:

.. code-block:: bash

    cucumber --tags ~@todo&&~@wip

You can use this tag logic in your :doc:`../behat/hooks` as well.
