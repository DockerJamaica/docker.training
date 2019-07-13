.. Docker Training documentation master file, created by
   sphinx-quickstart on Fri July  13 06:02:52 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Robot Frameowrk Docker
==========================
The Robot Framework is a generic test automation framework for acceptance 
testing and acceptance test-driven development. It is a keyword-driven testing
framework that uses tabular test data syntax.


Normally, the requirements for running the Django DRF application are:
* Python 2.7 or greater
* Python Virtualenv or PyEnv


Why use Docker with the Robot Frameowrk
--------------------------------------------------------------------

* Rapidly get started with automated testing without additional setup
* Allows testers to focus on test case and not the executing environment
* Isolating dependencies and source code:
* Remove the need of having multiple version of python installed on the host machine.
* Remove version conflicts of python packages
* It works on everyone’s machine. Docker eliminates the “but it worked on my laptop” problem.


Getting Started with Docker Robot Frameowrk
====================================================

Clone this repository

::

   git clone git@github.com:ypasmk/robot-framework-docker.git

Pull the image.

::

   docker pull ypasmk/robot-framework

Run the tests example tests

::

   cd robot-framework-docker && ./run_tests.sh


Contents
========

This image contains the following to facilitate robot testing

Xvfb
----

You can use it to start a visual display and fire up a browser for UI
testing.

Example (suites/virtual_display.robot):

::

   Start Virtual Display    1920    1080

Selenium2Library
----------------

More details here
http://robotframework.org/Selenium2Library/Selenium2Library.html

Also have a look at **suites/virtual_display.robot**

HttpLibrary.HTTP
----------------

More details here https://github.com/peritus/robotframework-httplibrary

Example:

::

   Create Http Context     api.some-end-point.com
   GET                     /some/service/that/supports/get
   Verify Status           200
   ${response}=            Get Response Body
   [return]                ${response}

robotframework-sshlibrary
-------------------------

More details here
http://robotframework.org/SSHLibrary/latest/SSHLibrary.html

robotframework-excellibrary
---------------------------

More details here
http://navinet.github.io/robotframework-excellibrary/ExcelLibrary-KeywordDocumentation.html

**ENJOY**

*For any requests or changes please open issues or create pull requests
:)*



Sources:
+++++++++

* `Robot Framework With Docker in less than 10 minutes by Ipatios Asmanidis <https://medium.com/@ypasmk/robot-framework-with-docker-in-less-than-10-minutes-7b86df769c22>`
* `robot-framework-docker <https://github.com/ypasmk/robot-framework-docker/>`


.. toctree::
   :maxdepth: 2
   :caption: React Training: