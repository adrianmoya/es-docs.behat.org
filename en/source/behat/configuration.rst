Configuration
=============

From version 1.1 behat has very powerful configuration system with which you can setup different test profiles and configure Behat from start to the end.

Default behat.yml
-----------------

By default, behat will look for file called ``behat.yml`` or ``config/behat.yml`` in current working directory. This file (if exists) is configuration file for behat.

Custom configuration file
-------------------------

But if you want to specify custom configuration file (``behat.yml.dist`` for example), you can do it with ``--config`` parameter (or it's ``-c`` shortcut):

.. code-block:: bash

    behat --config behat.yml.dist

Configuration
-------------

All configuration parameters are defined under profile name root (``default:`` for example). Profile is just a custom name, which you can use to fastly switch testing configuration throught ``--profile`` parameter.

Default profile is ``default``. All other profiles inherit parameters from ``default`` one. If you don't need any profile other than default - define your parameters as follows:

.. code-block:: yaml
    
    # behat.yml
    default:
        ...

Paths
~~~~~

First configuration block is ``paths``. Parameters under this configuration block tells behat where to find your features, definitions and support files:

.. code-block:: yaml
    
    # behat.yml
    default:
        paths:
            features:   '%%BEHAT_BASE_PATH%%'
            support:    '%behat.paths.features%/support'
            steps:
                - '%behat.paths.features%/steps'
            steps_i18n:
                - '%behat.paths.features%/steps/i18n'
            bootstrap:
                - '%behat.paths.support%/bootstrap.php'
            environment:
                - '%behat.paths.support%/env.php'
            hooks:
                - '%behat.paths.support%/hooks.php'

* ``features`` parameter defines where behat will look for your ``*.feature`` files.
* ``support`` parameter defines where behat will look for your support files (``env.php``, ``hooks.php`` and others).
* ``steps`` defines where behat will look your step definitions.
* ``steps_i18n`` defines where behat will look your step definitions translations (in xliff).
* ``bootstrap`` defines list of botstraping scripts.
* ``environment`` defines list of environment configuration files.
* ``hooks`` defines hooks definition files list.

As you can see, ``steps``, ``steps_i18n``, ``bootstrap``, ``environment`` and ``hooks`` parameters accepts arrays. It means, that behat can look inside multiple folders/paths to find proper step definition or environment object if you need so.

Also, notice the ``%behat.paths.features%`` and ``%%BEHAT_BASE_PATH%%`` placeholders. These strings are predefined configuration variables and constants, that you can use to build very flexible configurations:

* A placeholder that consists of upper-case letters and starts/ends with ``%%`` is a constant. Default constants are:
    1. ``%%BEHAT_BASE_PATH%%`` - current base path (specified as ``feature`` argument to behat command).
    2. ``%%BEHAT_WORK_PATH%%`` - current work path, from which behat was called.
    3. ``%%BEHAT_CONFIG_PATH%%`` - configuration file directory path.
* A placeholder that consists of lower-case letters and starts/ends with single ``%`` is a variable. These variables are your current configuration parameters, which you can use to nest configurations. Usable variables are:
    1. ``%behat.paths.features%`` - default or specified by you features path.
    2. ``%behat.paths.support%`` - default or specified by you support path.

Filters
~~~~~~~

Another very useful configuration block is ``filters`` block. This block defines default filters (name or tag) for your features. It means, that if you type same filters again and again from run to run, then it would be better if you define them as parameters:

.. code-block:: yaml

    # behat.yml
    default:
        filters:
            tags:   '@wip'

This filter parameters (``name`` and ``tags``) accept same strings as behat ``--name`` or ``--tags`` params does.

Formatter
~~~~~~~~~

If you need to customize your output formatter - ``formatter`` block is right for you:

.. code-block:: yaml

    # behat.yml
    default:
        formatter:
            name:                   'pretty'
            decorated:              true
            verbose:                false
            time:                   true
            language:               'en'
            output_path:            null
            multiline_arguments:    true
            parameters:
                ...

* ``name`` defines output formatter name to use for your features by default. You can write class name here, so behat will use your custom class as default output formatter, but be careful - this class should be accessible by behat and implement `FormatterInterface <http://docs.behat.org/api/behat/behat/behat/formatter/formatterinterface.html>`_, in other case behat will throw exception.
* ``decorated`` defines whether you want to use colors in your output. That's simple.
* ``verbose`` defines whether you want to see full exception stack traces in your output or not.
* ``time`` defines whether you want to see timer at the end of your formatter outpuy.
* ``language`` defines formatter language.
* ``output_path`` defines output path. If this parameters specified - behat will write your features into specified path instead of stdout.
* ``multiline_arguments`` defines whether you want to see multiline arguments (tables and pystrings) in pretty formatter output.
* ``parameters`` define list of additional parameters, that can be supported by specified custom formatter. But be careful - default formatters will throw exceptions if you'll try to pass unsupported parameter to them.

Environment
~~~~~~~~~~~

``env.php`` is very powerful configuration mechanism. But sometimes, it's not enough and you need to use your own environment class instead of default one or pass some environment parameters through ``behat.yml`` configuration. That's what ``environment`` block for:

.. code-block:: yaml

    # behat.yml
    default:
        environment:
            class:              'Behat\Behat\Environment\Environment'
            parameters:
                start_url:      'http://test.mink.loc'

* ``class`` defines which class you want to use as environment. This class should be accessible by behat and implement `EnvironmentInterface <http://docs.behat.org/api/behat/behat/behat/environment/environmentinterface.html>`_.
* ``parameters`` parameters is a simple parameters hash, that will be passed into environment ``setParameter`` setters. You can get environment parameters with ``getParameter`` call on environment object.

Profiles
--------

Sometimes, you might need to define different test running profiles for your test suite. Profiles will help you with that. Let's say we need 2 different profiles, that share common configurations, but use different formatters. Our ``behat.yml`` will looks like that:

.. code-block:: yaml

    # behat.yml
    default:
        environment:
            class:      'Your\Custom\EnvironmentClass'
    wip:
        filters:
            tags:       '@wip'
        formatter:
            name:       'progress'
    ci:
        formatter:
            name:        'junit'
            output_path: '/var/temp/junit'

This file defines 2 additional profiles (additional to default). Every profile will use ``Your\Custom\EnvironmentClass`` as environment object, but ``wip`` profile will run only scenarios with WorkInProgress (``@wip``) tag and will output them with ``progress`` formatter and ``ci`` will run all your features and output them with ``junit`` formatter into ``/var/temp/junit`` path. That simple!

To run custom profile, use ``--profile`` parameter:

.. code-block:: bash

    behat --profile wip
    behat --profile ci

Imports
-------

Sometimes you might want to share your configuration between different projects and their test suites. ``imports`` block comes to rescue:

.. code-block:: bash

    # behat.yml
    imports:
        - 'some_installed_pear_package_or_lib/behat.yml'
        - '/full/path/to/custom_behat_config.yml'

All files from ``imports`` block will be loaded by behat and merged into your ``behat.yml`` configs.
