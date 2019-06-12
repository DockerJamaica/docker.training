.. Docker Training documentation master file, created by
   sphinx-quickstart on Wed Jun  12 12:02:52 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Known Issues and Fixes
====================================================

The code for the landing page at https://dockertraining.readthedocs.io is in another repository: https://github.com/JamaicanDevelopers/docker.training.


If you are having issues, please let us know by opening an issue in our `Issue Tracker <https://github.com/JamaicanDevelopers/docker.training/issues>`_ or asking a question on our
`Community Space <https://jamaicandevelopers.com/Docker>`_.


Docker Toolbox issues - Windows 10 Home
-------------------------------------------------

Docker Toolbox runs Docker engine on top of boot2docker VM image running in Virtualbox hypervisor.
We can run Linux containers on top of the Docker engine.
We can run Docker Toolbox on any Windows variants.

localhost:8080 is not working
++++++++++++++++++++++++++++++++
The localhost address does not work when accessing the Django app from browser when using Docker Toolbox on Windows.

**Solution:** Unlike Docker Desktop, Dooker Toolbox uses Virtual Machine to run its Docker engine. As a result,
the Docker Toolbox VM maybe behind a  NAT address. Please check your Virtual Machine for Docker and use the IP address for the VM.
In my case, 192.168.99.100 worked for me as stated in an issue found [here](https://forums.docker.com/t/cant-connect-to-container-on-localhost-with-port-mapping/52716/13),

Credits to Jhamali_ for this fix.


Docker Desktop issues - Windows 10 Pro
-------------------------------------------

Docker for Windows requires that you’re running Windows Pro, Enterprise, or Education edition. Docker for Windows does not requires a
Virtual Machine to run Docker containers. All containers are executed on the host kernel.

docker exec -it django_drf_app /bin/bash" on Win 10 Pro not working
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Executing the following command `docker exec -it django_drf_app /bin/bash` results in an error using Docker Desktop on Windows 10 Pro.

**Solution:** Run the following command instead: `winpty docker exec -it django_drf_app bash`

Credits to Don-1_


.. _Jhamali: https://github.com/Jhamali
.. _Don-1: https://github.com/Don-1