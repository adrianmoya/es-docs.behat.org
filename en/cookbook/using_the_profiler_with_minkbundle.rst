Using the Symfony2 Profiler with Behat and MinkBundle
=====================================================

Accessing the Symfony2 profiler can be useful to test some parts of your
web application that does not hit the browser. Assuming it is supposed to
send an email, you can use the profiler to test that the email is sent correctly.

Your goal here will be to implement a step like this:

.. code-block:: gherkin

     Then I should get an email on "stof@example.org" containing "To finish validating your account - please visit"

.. note:

    You can only access the profiler when using the SymfonyDriver which gives
    you access to the kernel handling the request. You will need to tag your
    scenario so that the `symfony` session is used.

    .. code-block:: gherkin

        @mink:symfony
        Scenario: I should receive an email

First, retrieve the driver and check that it is the good one (useful when
you let someone else write the features with this step to avoid misuses)
and that the profiler is enabled:

.. code-block:: php

    <?php

    namespace Knp\AcademyBundle\Features\Context;

    use Behat\BehatBundle\Context\MinkContext;
    use Behat\Behat\Exception\PendingException;
    use Behat\MinkBundle\Driver\SymfonyDriver;
    use Behat\Mink\Exception\UnsupportedDriverActionException;
    use Behat\Mink\Exception\ExpectationException;

    /**
     * Feature context.
     */
    class FeatureContext extends MinkContext
    {
        /**
         * @Given /^(?:|I )should get an email on "(?P<email>[^"]+)" containing "(?P<text>(?:[^"]|\\")*)"$/
         */
        public function iShouldGetAnEmail($email, $text)
        {
            $driver = $this->getSession()->getDriver();
            if (!$driver instanceof SymfonyDriver) {
                throw new UnsupportedDriverActionException('You need to tag the scenario with "@mink:symfony". Using the profiler is not supported', $driver);
            }

            $profile = $driver->getClient()->getProfile();
            if (false === $profile) {
                throw new \RuntimeException('Emails cannot be tested as the profiler is disabled.');
            }

            throw new PendingException();
        }
    }

It is now time to use the profiler to implement the logic of the step:

.. code-block:: php

    <?php

    namespace Knp\AcademyBundle\Features\Context;

    use Behat\BehatBundle\Context\MinkContext;
    use Behat\Behat\Exception\PendingException;
    use Behat\MinkBundle\Driver\SymfonyDriver;
    use Behat\Mink\Exception\UnsupportedDriverActionException;
    use Behat\Mink\Exception\ExpectationException;

    /**
     * Feature context.
     */
    class FeatureContext extends MinkContext
    {
        /**
         * @Given /^(?:|I )should get an email on "(?P<email>[^"]+)" containing "(?P<text>(?:[^"]|\\")*)"$/
         */
        public function iShouldGetAnEmail($email, $text)
        {
            // the previous checks
            $expected = str_replace('\\"', '"', $text);
            $error = sprintf('No message sent to "%s"', $email);

            // Retrieving the swiftmailer collector to access the sent messages.
            foreach ($profile->getCollector('swiftmailer')->getMessages() as $message) {
                /* @var $message \Swift_Message */
                if (!array_key_exists($email, $message->getTo()) &&
                    !($message->getHeaders()->has('X-Swift-To') && array_key_exists($email, $message->getHeaders()->get('X-Swift-To')->getFieldBodyModel()))
                ) {
                    // Checking the recipient email and the X-Swift-To header to handle the the RedirectingPlugin
                    // If the recipient is not the expected one, check the next mail
                    continue;
                }

                try {
                    // checking the content
                    return assertContains($expected, $message->getBody());
                } catch (\PHPUnit_Framework_ExpectationFailedException $e) {
                    $error = sprintf('An email has been found for "%s" but without the text "%s".', $email, $expected);
                }
            }

            throw new ExpectationException($error, $this->getSession());
        }
    }
