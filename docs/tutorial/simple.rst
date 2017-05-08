.. _tutorial-simple:

############
introduction
############

Lettuce_ is an extremely useful and charming tool for BDD_ (Behavior
Driven Development). It can execute plain-text functional descriptions
as automated tests for Python_ projects, just as Cucumber_ does for
Ruby_.

Lettuce_ makes the development and testing process really easy,
scalable, readable and - what is best - it allows someone who doesn't
program to describe the behavior of a certain system, without
imagining those descriptions will automatically test the system during
its development.

.. image:: ./flow.png

***********
get lettuce
***********

Make sure you've got Python installed and then run from the terminal:

.. highlight:: bash

::

   user@machine:~$ [sudo] pip install lettuce

****************
define a problem
****************

Let's choose a problem to lettuce:
**Given a number, what is its factorial?**

.. Note::

   The factorial of a positive integer n, denoted by n!, is the
   product of all positive integers less than or equal to n. The
   factorial of 0 is 1

*****************
project structure
*****************

Build the directory tree bellow such as the files ``zero.feature`` and ``steps.py`` are empty.

.. highlight:: bash

::

    /home/user/projects/mymath
         | tests
               | features
                    - zero.feature
                    - steps.py

***********
lettuce it!
***********

Lets begin to describe and solve our problem...

first round
===========


[a] describe behaviour
----------------------

Start describing the expected behaviour of factorial in ``zero.feature`` using English:

.. highlight:: ruby

::

    Feature: Compute factorial
        In order to play with Lettuce
        As beginners
        We'll implement factorial

        Scenario: Factorial of 0
            Given I have the number 0
            When I compute its factorial
            Then I see the number 1

.. Note::

    zero.feature must be inside features directory and its extension must
    be .feature. However, you're free to choose its name.

[b] define steps in python
--------------------------

Now let's define the steps of the scenario, so Lettuce can
understand the behaviour description. Create the ``steps.py`` file which will contain
python code describing the steps.

Python:

.. highlight:: python

::

   from lettuce import *

   @step('I have the number (\d+)')
   def have_the_number(step, number):
       world.number = int(number)

   @step('I compute its factorial')
   def compute_its_factorial(step):
       world.number = factorial(world.number)

   @step('I see the number (\d+)')
   def check_number(step, expected):
       expected = int(expected)
       assert world.number == expected, \
           "Got %d" % world.number

   def factorial(number):
       return -1

.. Note::

   ``steps.py`` must be inside features directory, but the names doesn't
   need to be ``steps.py`` it can be any python file with a ``.py`` extension.
   Lettuce_ will look for python files recursively within features
   dir.

Ideally, factorial will be defined somewhere else. However, as this is
just a first example, we'll implement it inside steps.py, so you get
the idea of how to use Lettuce.

**Notice that, until now, we haven't defined the factorial function (it's returning -1).**

[c] run and watch it fail
-------------------------

Go to the tests directory and run from the terminal:

.. highlight:: bash

::

   user@machine:~/projects/mymath/tests$ lettuce

As you haven't implemented factorial, it is no surprise the behavior
won't be reached:

.. image:: ./screenshot1.png

Our only scenario failed :(
Let's solve it...

[d] write code to make it pass
------------------------------

Well, by definition, we know that the factorial of 0 is 1. As our only
feature is this... we could force factorial to return 1.

.. highlight:: python

::

    from lettuce import *

    @step('I have the number (\d+)')
    def have_the_number(step, number):
        world.number = int(number)

    @step('I compute its factorial')
    def compute_its_factorial(step):
        world.number = factorial(world.number)

    @step('I see the number (\d+)')
    def check_number(step, expected):
        expected = int(expected)
        assert world.number == expected, \
            "Got %d" % world.number

    def factorial(number):
        return 1

[e] run again and watch it pass
-------------------------------

Again, run from the terminal:

.. highlight:: bash

::

   user@machine:~/projects/mymath/tests$ lettuce

And you'll be happy to see your factorial implementation passed all the behaviours expected:

.. image:: ./screenshot2.png

Great! :)

However, one test is not enough for checking the quality of our
solution... So let's lettuce it again!

second round
============

Let's provide more tests so our problem is better described, and so we
provide a more accurate implementation of factorial:

[a] describe behaviour
----------------------

Let's provide two new scenarios, for numbers 1 and 2:

.. highlight:: ruby

::

    Feature: Compute factorial
      In order to play with Lettuce
      As beginners
      We'll implement factorial

      Scenario: Factorial of 0
        Given I have the number 0
        When I compute its factorial
        Then I see the number 1

      Scenario: Factorial of 1
        Given I have the number 1
        When I compute its factorial
        Then I see the number 1

      Scenario: Factorial of 2
        Given I have the number 2
        When I compute its factorial
        Then I see the number 2

[b] define steps in python
--------------------------

As we haven't changed the definition, no need to make changes on this
step.

[c] run and watch it fail
-------------------------


.. highlight:: bash

::

   user@machine:~/projects/mymath/tests$ lettuce

When running Lettuce we realize that our previous implementation of
factorial works fine both for 0 and for 1, but not for 2 - it
fails. :(

.. image:: ./screenshot3.png

[d] write code to make it pass
------------------------------

Let's provide a solution so we get the right factorial for all
scenarios, specially for number 2:

.. highlight:: python

::

    from lettuce import *

    @step('I have the number (\d+)')
    def have_the_number(step, number):
        world.number = int(number)

    @step('I compute its factorial')
    def compute_its_factorial(step):
        world.number = factorial(world.number)

    @step('I see the number (\d+)')
    def check_number(step, expected):
        expected = int(expected)
        assert world.number == expected, \
            "Got %d" % world.number

    def factorial(number):
        number = int(number)
        if (number == 0) or (number == 1):
            return 1
        else:
            return number

[e] run again and watch it pass
-------------------------------

.. highlight:: bash

::

   user@machine:~/projects/mymath/tests$ lettuce

.. image:: ./screenshot4.png

Great! Three scenarios described and they are alright!

third round
===========

Let's provide more tests so our problem is better described and we get
new errors so we'll be able to solve them.

[a] describe behaviour
----------------------

.. highlight:: ruby

::

    Feature: Compute factorial
      In order to play with Lettuce
      As beginners
      We'll implement factorial

      Scenario: Factorial of 0
        Given I have the number 0
        When I compute its factorial
        Then I see the number 1

      Scenario: Factorial of 1
        Given I have the number 1
        When I compute its factorial
        Then I see the number 1

      Scenario: Factorial of 2
        Given I have the number 2
        When I compute its factorial
        Then I see the number 2

      Scenario: Factorial of 3
        Given I have the number 3
        When I compute its factorial
        Then I see the number 6

      Scenario: Factorial of 4
        Given I have the number 4
        When I compute its factorial
        Then I see the number 24

[b] define steps in python
--------------------------

As we haven't changed the definition, no need to make changes on this
step.

[c] run and watch it fail
-------------------------

.. highlight:: bash

::

   user@machine:~/projects/mymath/tests$ lettuce

.. image:: ./screenshot5.png

[d] write code to make it pass
------------------------------

.. highlight:: python

::

    from lettuce import *

    @step('I have the number (\d+)')
    def have_the_number(step, number):
        world.number = int(number)

    @step('I compute its factorial')
    def compute_its_factorial(step):
        world.number = factorial(world.number)

    @step('I see the number (\d+)')
    def check_number(step, expected):
        expected = int(expected)
        assert world.number == expected, \
            "Got %d" % world.number

    def factorial(number):
        number = int(number)
        if (number == 0) or (number == 1):
            return 1
        else:
            return number*factorial(number-1)

[e] run again and watch it pass
-------------------------------

.. highlight:: bash

::

   user@machine:~/projects/mymath/tests$ lettuce

.. image:: ./screenshot6.png

forth round
===========

All steps should be repeated as long as you can keep doing them - the
quality of your software depends on these.

****************
Syntactic sugar
****************

Available for versions > 0.2.19

Steps sentence can now be given by function name or doc.
=========================================================

To take a step sentence from function name or doc,
just decorate it with "@step" without argument.

These two steps below, are identicals than the example above.

.. highlight:: python

::

    from lettuce import *

    @step
    def have_the_number(step, number):
        'I have the number (\d+)'
        world.number = int(number)

    @step
    def i_compute_its_factorial(step):
        world.number = factorial(world.number)



Steps can be grouped in class decorated with "@steps"
======================================================

.. highlight:: python

::

    from lettuce import world, steps

    @steps
    class FactorialSteps(object):
      """Methods in exclude or starting with _ will not be considered as step"""

      exclude = ['set_number', 'get_number']

      def __init__(self, environs):
        self.environs = environs

      def set_number(self, value):
        self.environs.number = int(value)

      def get_number(self):
        return self.environs.number

      def _assert_number_is(self, expected, msg="Got %d"):
          number = self.get_number()
          assert number == expected, msg % number

      def have_the_number(self, step, number):
        '''I have the number (\d+)'''
          self.set_number(number)

      def i_compute_its_factorial(self, step):
          number = self.get_number()
          self.set_number(factorial(number))

      def check_number(self, step, expected):
          '''I see the number (\d+)'''
          self._assert_number_is(int(expected))

    # Important!
    # Steps are added only when you instanciate the "@steps" decorated class
    # Internally decorator "@steps" build a closure with __init__

    FactorialSteps(world)

    def factorial(number):
        number = int(number)
        if (number == 0) or (number == 1):
            return 1
        else:
            return number*factorial(number-1)


Have a nice lettuce...! ;)
