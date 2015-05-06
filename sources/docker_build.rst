:title: Custom Containers with Docker Build
:description: Running minions in a Docker container defined by a Dockerfile
:keywords: shippable, Docker, Container

.. _docker_build:

.. note::
  Docker Build Support is with dedicated hosts only!

DOCKER SUPPORT
==============
Shippable is the world's only CI/CD platform built natively on Docker. All builds are run on Docker containers and this gives us a unique ability to support advanced docker workflows. We're constantly adding to our custom Docker support, so check back often!

**Connect your Docker Hub account to Shippable**
------------------------------------------------
If you want to interact with Docker Hub in any part of your build workflow, you need to connect your Docker Hub account to Shippable. This is a requirement for pulling custom images from your Docker Hub repos or pushing images to Docker Hub.

You can set up Docker Hub integration on a per-Org basis.

1. Go to your Organization's page on Shippable. You can find your Organization name at the right side of your Dashboard and click on it to go to the Org page.
2. If the 'Docker Hub' icon at the top right of your Org page is set to ON, you have already connected your Docker Hub account to your Org. If it is set to OFF, click on it, enter your Docker Hub credentials, and click on Save.

We do not call the Docker Hub API until we need to do so during an actual build, so we have no way of knowing if your credentials are correct at this point. The 'Docker Hub' icon will turn green after you save your creds, but if you run into problems with login to Docker Hub during the build, you should check your creds again.

-------
