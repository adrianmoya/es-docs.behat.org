Step Argument Transformations
=============================

Step argument transformations help your step definitions be more DRY. You can use the ``$steps->Transform`` method to register a regular expression along with a callback. Before captured Regexp groups get passed as arguments to step definitions, an attempt is made to match them against any registered ``Transform``. If there is a match, the return value of the callback supplied with the transform is passed as the step definition argument instead.

It’s recommended that you put ``Transform`` code in the same file as the step definitions using them. It becomes easier to navigate your code that way. Here is an example:

.. code-block:: php

    <?php
    // features/steps/trans_steps.php

    $steps->Transform('/^user \w+$/', function($arg) {
        preg_match('/\w+$/', $arg, $matches);

        return new User($matches[0]);
    });

    $steps->Then('/^(user \w+) should be friends with (user \w+)$/', function($user, $friend) {
        assertTrue($user->isFriendOf($friend));
    });

.. code-block:: gherkin

    Feature:

        Scenario: befriending
          Given ...
          When ...
          Then user larrytheliquid should be friends with user dastels

Note that sometimes you may not need to parse the entire arg and may only care about a specific Regexp capture group being matched. For this more common case, you can use an optional syntax for Transform where the block supplied has arguments that correspond to capture groups in the registered Regexp:

.. code-block:: php

    <?php
    // features/steps/trans_steps.php

    $steps->Transform('/^user (\w+)$/', function($username) {
        return new User($username);
    });

.. note
    Remember is to provide some contextual data like “user” so you don’t end up transforming every single argument to a step definition.

Transforming tables
-------------------

Step argument transformations can also be used with tables. A table is matched via a comma-delimited list of the column headers and the ‘table:’ prefix:

.. code-block:: gherkin

    Feature:

        Scenario: setting up via table
          Given ...
          When ...
          Then I should have
            | name  | age |
            | corey | 36  |

.. code-block:: php

    <?php
    // features/steps/trans_steps.php

    $steps->Transform('/^table:name,age$/', function($table) {
        return $table->getHash()
    });
