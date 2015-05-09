:title: Custom Containers with Docker Build
:description: Running minions in a Docker container defined by a Dockerfile
:keywords: shippable, Docker, Container

.. _docker_build:

.. note::
  Docker Build Support is with dedicated hosts only!

DOCKER REGISTRIES
=================
Shippable is the world's only CI/CD platform built natively on Docker. All builds are run on Docker containers and this gives us a unique ability to support advanced docker workflows. We're constantly adding to our custom Docker support, so check back often!

We fully integrate with 2 hosted Docker registries - Docker Hub and Google Container Registry.

**Docker Hub**
--------------

**Connect your Docker Hub account to Shippable**

If you want to interact with Docker Hub in any part of your build workflow, you need to connect your Docker Hub account to Shippable. This is a requirement for pulling custom images from your Docker Hub repos or pushing images to Docker Hub.

You can set up Docker Hub integration on a per-Org basis.

1. Go to your Organization's page on Shippable. You can find your Organization name at the right side of your Dashboard and click on it to go to the Org page.
2. If the 'Docker Hub' icon at the top right of your Org page is set to ON, you have already connected your Docker Hub account to your Org. If it is set to OFF, click on it, enter your Docker Hub credentials, and click on Save.

We do not call the Docker Hub API until we need to do so during an actual build, so we have no way of knowing if your credentials are correct at this point. The 'Docker Hub' icon will turn green after you save your creds, but if you run into problems with login to Docker Hub during the build, you should check your creds again.

-------

**Push to Docker Hub**

Shippable allows you to push an image to the docker registry after a successful build. To do this, make sure your Docker Hub icon is set to ON on your Organization's page on Shippable.

The following configuration in your shippable.yml will push the image to Docker Hub after the build is successful.

.. code-block:: bash

    commit_container: username/sample_project

The username above should be the same as the Docker Hub credentials you entered in the previous step.

-------

**Dockerbuild**

.. note::
  Docker Build Support is only available with dedicated hosts. To set up a dedicated host, please follow instructions `here <http://docs.shippable.com/en/latest/config.html#dedicated-hosts>`_

You can run your build in a custom docker container by building a Docker image from a Dockerfile. Aside from providing a custom environment for your build, this image created can be pushed to your Docker Hub account, for later use in your deployment step.

There are 2 ways to set up Docker build with Shippable - pre CI or post CI. 

Pre CI workflow is:
* Build the image using Dockerfile at the root of your repo
* Pull code from GitHub/Bitbucket and test code in the container
* Push container to docker hub

Post CI workflow is:
* Pull image specified from Docker Hub (default is minv2)
* Pull code from GitHub/Bitbucket and test in container
* If CI passs, build container from Dockerfile at the root of the repo
* Push container to docker hub

To use these workflows, your app must be "dockerized". Details on this can be found in Docker's official documentation `Docker's official documentation <https://docs.dockerhub.com>`_. You can also look at our `Docker build sample app <https://github.com/cadbot/dockerized-nodejs>`_. 

**Pre CI Dockerbuild**

* Make sure your Shippable org is connected to your Docker Hub account
* Enable the repository on Shippable
* On the repo page, go to 'Settings'. Choose the following -

  * Build image : Custom Image
  * Custom image action : Build
  * Custom image name : (docker hub username)/(image name)
  * Source code path : (source code path for image you want to build)
  * Push to Docker Hub : Check
* Make sure the Dockerfile for the image you want to build is at the root of your repo
* Trigger a manual or webhook build
* After the build is complete, make sure your Docker Hub account has the image you just pushed. The image should be tagged with the build number on Shippable.

**Post CI Dockerbuild**

* Make sure your Shippable org is connected to your Docker Hub account
* Enable the repository on Shippable
* On the repo page, go to 'Settings'. Choose the following -

  * Build image : Custom Image
  * Custom image action : Build
  * Custom image name : (docker hub username)/(image name)
  * Source code path : (source code path for image you want to build)
  * Docker build when finished : Check
  * Image to pull: Specify image you want to run tests on, default is shippable/minv2
  * Push to Docker Hub : Check
* Make sure the Dockerfile for the image you want to build is at the root of your repo
* Trigger a manual or webhook build
* After the build is complete, make sure your Docker Hub account has the right image. The image should be tagged with the build number on Shippable.

**Copying artifacts to prod image**

If you are following the post-CI Dockerbuild workflow and  want to copy some build artifacts to your prod image, you should-

1. Create a shippable/buildoutput directory in your shippable.yml

.. code-block:: bash

  before_script:
    - mkdir -p shippable/buildoutput

2. In the after_script section, copy whatever you want to this directory

.. code-block:: bash

  after_script:
    - cp -r (your artifacts) ./shippable/buildoutput

3. In your Dockerfile, you can now use ADD to put the artifacts wherever you want in your prod image

.. code-block:: bash

  ADD ./buildoutput/(artifacts file) (target)

And that's it. Any artifacts you need will be available in your prod image.


**Google Container Registry**
-----------------------------
The Google Container Registry (GCR) provides secure, private Docker image storage on Google Cloud Platform. Using GCR has many advantages such as fine grained access control, server-side encryption of images, and super fast deployment to Google Container Engine and Google Compute Engine.

To read more about GCR, you can read their `documentation <https://cloud.google.com/tools/container-registry/>`_ or their `announcement blog <http://googlecloudplatform.blogspot.com/2015/01/secure-hosting-of-private-Docker-repositories-in-Google-Cloud-Platform.html>`_ 

**Setting up GCR integration on Shippable**

If you want to interact with GCR in any part of your build workflow for your Shippable project, such as using your private images for your builds or pushing images to your repository, you need to connect your GCR project to your Shippable project. 

Follow the following steps to set up GCR integration.

1. Create a Project in Google Dev Console

To use Shippable with GCR, you will need a project created using using the Google Developers Console (GDC). According to their documentation - A project is a collection of settings, credentials, and metadata about the application or applications you're working on that make use of Google Developer APIs and Google Cloud resources.

If you already have a project you want to use, skip to step 2.

To create a project -

* Sign in to the `Google Developers Console <https://console.developers.google.com/>`_ 
* Click on 'Create Project'
* Enter a name and project ID or accept the defaults.
* Click 'Create'

2. Setting up OAuth for your GDC project

* On the `Google Developers Console <https://console.developers.google.com/>`_ , select the project you just created
* In the sidebar on the left, expand 'APIs & auth' and select 'Credentials'
* Click 'Create new Client ID' and select 'Service Account' in the pop-up window
* Click on 'Create Client ID'. A dialog box appears. To proceed, click 'Okay, got it'
* Your new Public/Private key pair is generated and downloaded to your machine. Please store this carefully since you will not be able to retrieve this from your GDC account. You will need this key pair to set up GCR integration on Shippable. 

3. Set up GCR Integration 

* Login to Shippable
* Click on your GitHub/Bitbucket username at the top right of your Dashboard and click on 'Account Settings'
* On the Account Settings page, click on 'Integrations', just below the Account Settings text.
* From the options presented, click on GCR
* Enter an Integration name, which will be used to refer to this integration on Shippable
* Copy the key pair generated during the last step and paste into the jsonKey field.
* Click on 'Save'

At this point, you have set up GCR integration at an account level. To push and pull from GCR, you will also need to enable repo-level access as described in the scenarios below.

-------

**Pull custom image from GCR**

Shippable allows you to pull a custom image from GCR to run your builds on. 

To enable GCR integration for the repository for which you want to pull a custom image -

* Go to your repository page on Shippable and click on 'Integrations' on the right sidebar
* Click on the dropdown for 'Hub' and select the Integration name you want to use.

The following configuration in your shippable.yml will pull your image from GCR and run your builds in the container -

.. code-block:: bash

    build_image: gcr.io/project_ID_on_GDC/image_name

-------

**Push to GCR**

Shippable allows you to push an image to GCR after a successful build. 

To enable GCR integration for the repository for which you want to push to GCR -

* Go to your repository page on Shippable and click on 'Integrations' on the right sidebar
* Click on the dropdown for 'Hub' and select the Integration name you want to use.

The following configuration in your shippable.yml will push the image to GCR after the build is successful.

.. code-block:: bash

    commit_container: gcr.io/project_ID_on_GDC/image_name

-------

**Dockerbuild**

.. note::
  Docker Build Support is only available with dedicated hosts. To set up a dedicated host, please follow instructions `here <http://docs.shippable.com/en/latest/config.html#dedicated-hosts>`_

You can run your build in a custom docker container by building a Docker image from a Dockerfile. Aside from providing a custom environment for your build, this image created can be pushed to your GDC account, for later use in your deployment step.

There are 2 ways to set up Docker build with Shippable - pre CI or post CI. 

Pre CI workflow is:
* Build the image using Dockerfile at the root of your repo
* Pull code from GitHub/Bitbucket and test code in the container
* Push container to docker hub

Post CI workflow is:
* Pull image specified from Docker Hub (default is minv2)
* Pull code from GitHub/Bitbucket and test in container
* If CI passs, build container from Dockerfile at the root of the repo
* Push container to docker hub

To use these workflows, your app must be "dockerized". Details on this can be found in Docker's official documentation `Docker's official documentation <https://docs.dockerhub.com>`_. You can also look at our `Docker build sample app <https://github.com/cadbot/dockerized-nodejs>`_. 

**Pre CI Dockerbuild**

* Enable the repository on Shippable
* Make sure that GCR integration is set up on Shippable and that GCR is enabled for your repo
* On the repo page, go to 'Settings'. Choose the following -

  * Docker Build : ON
  * CI order : Pre-CI
  * Push Build : Yes if you want to push to GCR, No if you don't want to push to GCR 
  * Image name : gcr.io/(project id on GDC)/(image name)  
    We need an image name for the image we build from your Dockerfile, even if you choose not to push to GCR
  * Source Location : (source code location where tests will be run)
* Make sure the Dockerfile for the image you want to build is at the root of your repo
* Trigger a manual or webhook build
* After the build is complete, make sure your GDC account shows the image you just pushed. The image should be tagged with the build number on Shippable.

**Post CI Dockerbuild**

* Enable the repository on Shippable
* Make sure that GCR integration is set up on Shippable and that GCR is enabled for your repo
* On the repo page, go to 'Settings'. Choose the following -

  * Docker Build : ON
  * CI order : Post-CI
  * Push Build : Yes if you want to push to GCR, No if you don't want to push to GCR 
  * Image name : gcr.io/(project id on GDC)/(image name)  
  * Pull image name : Since your Dockerbuild is happening post CI, enter the image you want to use for CI

* Make sure the Dockerfile for the image you want to build is at the root of your repo
* Trigger a manual or webhook build
* After the build is complete, make sure your GDC account shows the image you just pushed. The image should be tagged with the build number on Shippable.

**Copying artifacts to prod image**

If you are following the post-CI Dockerbuild workflow and  want to copy some build artifacts to your prod image, you should-

1. Create a shippable/buildoutput directory in your shippable.yml

.. code-block:: bash

  before_script:
    - mkdir -p shippable/buildoutput

2. In the after_script section, copy whatever you want to this directory

.. code-block:: bash

  after_script:
    - cp -r (your artifacts) ./shippable/buildoutput

3. In your Dockerfile, you can now use ADD to put the artifacts wherever you want in your prod image

.. code-block:: bash

  ADD ./buildoutput/(artifacts file) (target)

And that's it. Any artifacts you need will be available in your prod image.
