Multiline Arguments
===================

The regular expression matching in :doc:`steps` lets you capture small strings from your
steps and receive them in your step definitions. However, there are times when you want to pass
a richer data structure from a step to a step definition.

This is what multiline step arguments are for. They are written on the lines right underneath a step,
and will be passed to definition callback as the last argument.

Multiline step arguments come in two flavours – tables or strings.

Tables
------

Tables as arguments to steps are handy for specifying a larger data set – usually as input to a Given or as expected output from a Then. (This is not to be confused with tables in :doc:`outlines` – syntactically identical, but with a different purpose). Example:

.. code-block:: gherkin

    Feature:

        Scenario:
            Given the following people exist:
              | name  | email           | phone |
              | Aslak | aslak@email.com | 123   |
              | Joe   | joe@email.com   | 234   |
              | Bryan | bryan@email.org | 456   |

and a matching step definition:

.. code-block:: php

    <?php
    // features/steps/name_steps.php

    $steps->Given('/^the following people exist:$/', function($world, $table) {
        $hash = $table->getHash();
        foreach ($hash as $row) {
            assertEquals($world->users[$row['name']]->getEmail(), $row['email']);
        }
    });

A plain text table gets stored in ``Behat\Gherkin\Node\TableNode`` instance, which provides several methods to access rows. Most common is ``->getHash()`` which returns hash representation of simple table and `->getRowsHash()`, which returns hash, created from rows.

Multiline Strings
-----------------

Multiline Strings are handy for specifying a larger piece of text. This is done using the PyString syntax. (The inspiration for PyString comes from Python where ``"""`` is used to delineate docstrings.) The text should be offset by delimiters consisting of three double-quote marks on lines of their own:

.. code-block:: gherkin

    Feature:

        Scenario:
            Given a blog post named "Random" with Markdown body
              """
              Some Title, Eh?
              ==============
              Here is the first paragraph of my blog post. Lorem ipsum dolor sit amet,
              consectetur adipiscing elit.
              """

In your step definition, there’s no need to find this text and match it in your Regexp. It will automatically be passed as the last argument into the step definition callback. For example:

.. code-block:: php

    <?php
    // features/steps/pystringarg_steps.php

    $steps->Given('/^a blog post named "(.*?)" with Markdown body$/', function($world, $title, $markdown) {
        Post::create(array('title' => $title, 'body' => (string) $markdown));
    });

.. note::
    PyStrings gets stored in PyStringNode instance, which you can simply convert to string with ``(string) $pystring`` as in example above.

Indentation of the opening ``"""`` is unimportant, although common practice is two spaces in from the enclosing step. The indentation inside the triple quotes, however, is significant. Each line of the string passed to the step definition’s callback will be de-indented according to the opening ``"""``. Indentation beyond the column of the opening ``"""`` will therefore be preserved.
