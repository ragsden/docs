:title: Custom Containers with Docker Build
:description: Running minions in a Docker container defined by a Dockerfile
:keywords: shippable, Docker, Container

.. _docker_build

.. note::
  Docker Build Support is with dedicated hosts only!

Docker Build Support
==========================
In addition to pointing to a Docker image on Docker Hub, you can also run your 
build in a custom docker container by instructing us to build a Docker image
from a Dockerfile. Aside from providing a custom environment for your build,
the image created can be pushed to your Docker Hub account, for later
use in your deployment step.

**Step 1: Setup Dedicated host**

Docker Build Support is for builds running on dedicated hosts only. To use this
feature you must first setup at least one dedicated host for your account.

**Step 2: Dockerfile and App Dockerization**

To use Docker Build with your builds, you must include a Dockerfile in the root directory of your app. Furthermore, this Dockerfile must be committed to the repo that you have configured Shippable to pull from, such as your team's Github repo.

In order for your build to run successfully, you must properly "dockerize" your application. Details on this can be found in Docker's official documentation `Docker's official documentation <https://docs.dockerhub.com>`_. You can also look at our `Docker build sample app <https://github.com/cadbot/dockerized-nodejs>`_. 

**Step 3: Enable Docker Build for your Project and Set Configurations** 

Finally, you must configure your app to use Docker Build through the Shippable Project Dashboard. To do this-

* Go to the Settings tab on your project's page and expand the Project settings option
* On the Build image dropdown menu, select 'Custom image'
* Select 'Build' on the Custom image action dropdown
* Specify a Custom image name and your Source code path. If you want to later push your image to Docker Hub, you should refer to the image by name you specified here. The source code path specifies where you have installed your app's source code on the running Docker container.
* Save settings

And that's it! For every build you run after this point, we will build your custom image from your Dockerfile, run CI, and push the container to Docker Hub.

Post CI Docker Build
------------------------
In addition to the above workflow, it is also possible to build a new image after your CI is finished. Doing this allows you to create a concise
docker image that contains only what you need for deployment, and leave out anything that is only required for building/testing. As there is no upfront
way for us to know which files you'd like to put in your "prod" docker image, you must manually specify which files or build artifacts you want to include in your prod image.

Please note that the post-CI Docker Build workflow  is not available if you are running a matrix build - i.e. if you are kicking off multiple builds for every code commit.

To start with, all of the above steps for regular Docker Build Support are a prerequisite; be sure all those steps are working first, before trying to debug
Post CI specific problems.

The first additional steps take place in your project's Settings tab on the Shippable web console. 
* Check the 'Docker build when finished' checkbox
* Enter your CI image in the 'image to pull' textbox. If you do not have an image for CI, you can use shippable/minv2. But please note that minv2 is a very big image so you might want to pull one of your own images for CI.

**Copying artifacts to prod image**
If you want to copy some build artifacts to your prod image, you should-
* create a shippable/buildoutput directory in your shippable.yml

.. code-block:: bash

  before_script:
    - mkdir -p shippable/buildoutput

* in the after_script section, copy whatever you want to this directory

.. code-block:: bash

  after_script:
    - cp -r (your artifacts) ./shippable/buildoutput

* In your Dockerfile, you can now use ADD to put the artifacts wherever you want in your prod image

.. code-block:: bash

  ADD ./buildoutput/(artifacts file) (target)

And that's it. Any artifacts you need will be available in your prod image.

If the 'Push to Docker Hub' option is checked, then this prod image will be built and pushed to Docker Hub. 
