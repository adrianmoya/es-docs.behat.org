Using the Symfony2 Profiler with Behat and MinkBundle
=====================================================

Accessing the Symfony2 profiler can be useful to test some parts of your
web application that does not hit the browser. Assuming it is supposed to
send an email, you can use the profiler to test that the email is sent correctly.

Your goal here will be to implement a step like this:

.. code-block:: gherkin

     Then I should get an email on "stof@example.org" containing:
        """
        To finish validating your account - please visit
        """

Bootstraping the Email Step
---------------------------

First, let's implement profiler retieving function which will check that the
current driver is the profilable one (useful when you let someone else write
the features with this step to avoid misuses) and that the profiler is enabled:

.. code-block:: php

    <?php

    namespace Acme\DemoBundle\Features\Context;

    use Behat\Gherkin\Node\PyStringNode;
    use Behat\Behat\Exception\PendingException;
    use Behat\BehatBundle\Context\MinkContext;

    use Behat\Mink\Exception\UnsupportedDriverActionException,
        Behat\Mink\Exception\ExpectationException;
    use Behat\MinkBundle\Driver\SymfonyDriver;

    /**
     * Feature context.
     */
    class FeatureContext extends MinkContext
    {
        public function getSymfonyProfiler()
        {
            $driver = $this->getSession()->getDriver();
            if (!$driver instanceof SymfonyDriver) {
                throw new UnsupportedDriverActionException(
                    'You need to tag the scenario with '.
                    '"@mink:symfony". Using the profiler is not '.
                    'supported by %s', $driver
                );
            }

            $profile = $driver->getClient()->getProfile();
            if (false === $profile) {
                throw new \RuntimeException(
                    'Emails cannot be tested as the profiler is '.
                    'disabled.'
                );
            }

            return $profile;
        }
    }

.. note::

    You can only access the profiler when using the SymfonyDriver which gives
    you access to the kernel handling the request. You will need to tag your
    scenario so that the `symfony` session is used.

    .. code-block:: gherkin

        @mink:symfony
        Scenario: I should receive an email

Implementing Email Step Logic
-----------------------------

It is now time to use the profiler to implement our email checking step:

.. code-block:: php

    /**
     * @Given /^(?:|I )should get an email on "(?P<email>[^"]+)" containing:$/
     */
    public function iShouldGetAnEmail($email, PyStringNode $text)
    {
        $profiler = $this->getSymfonyProfiler();
        $error    = sprintf('No message sent to "%s"', $email);

        // Retrieving the swiftmailer collector to access the
        // sent messages.
        $swiftmailerProfile = $profile->getCollector('swiftmailer');
        foreach ($swiftmailerProfile->getMessages() as $message) {
            $headers = $message->getHeaders();

            // Checking the recipient email and the X-Swift-To
            // header to handle the the RedirectingPlugin.
            // If the recipient is not the expected one, check
            // the next mail.
            $correctTo = array_key_exists(
                $email, $message->getTo()
            );
            $correctToHeader = $headers->has('X-Swift-To') &&
                array_key_exists(
                    $email,
                    $headers->get('X-Swift-To')->
                        getFieldBodyModel()
                );

            if (!$correctTo && !$correctToHeader) {
                continue;
            }

            try {
                // checking the content
                return assertContains(
                    $text->getRaw(), $message->getBody()
                );
            } catch (\PHPUnit_Framework_ExpectationFailedException $e) {
                $error = sprintf(
                    'An email has been found for "%s" but without '.
                    'the text "%s".', $email, $text->getRaw()
                );
            }
        }

        throw new ExpectationException($error, $this->getSession());
    }
