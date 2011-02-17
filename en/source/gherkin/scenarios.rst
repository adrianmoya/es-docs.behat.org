Scenarios
=========

Scenario is base Gherkin structure. Every scenario starts with ``Scenario:`` keyword (or localized one), followed or not with scenario title.
Every scenario consists of steps.

.. code-block:: gherkin

    Feature: Multiple site support
      As a Mephisto site owner
      I want to host blogs for different people
      In order to make gigantic piles of money

      Scenario: Dr. Bill posts to his own blog
        Given I am logged in as Dr. Bill
        When I try to post to "Expensive Therapy"
        Then I should see "Your article was published."

      Scenario: Dr. Bill tries to post to somebody else's blog, and fails
        Given I am logged in as Dr. Bill
        When I try to post to "Greg's anti-tax rants"
        Then I should see "Hey! That's not your blog!"

      Scenario: Greg posts to a client's blog
        Given I am logged in as Greg
        When I try to post to "Expensive Therapy"
        Then I should see "Your article was published."
