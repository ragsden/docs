:title: Getting Started 
:description: Basic getting started section
:keywords: getting started, questions, documentation, shippable

.. _getstarted:

Getting Started
===============


**Step 1** : Sign Up
--------------------

Using Shippable requires a Github or a Bitbucket account. To create a Shippable account with either of these credentials, visit `Shippable.com <https://www.shippable.com>`_ click **Login**, and choose if you'd like to use your Github or Bitbucket credentials.
If you have Github and Bitbucket accounts you'd like to integrate with shippable, come back and repeat the process with your other account.

If you signup with both Github and Bitbucket, you will have a separate Shippable account for each.

After entering your credentials, you will be prompted to give Shippable access to your repos.

.. note::

    We realize that most people do not want to give write access to their repo. However, we need write permissions to add deploy keys to your repos for our webhooks to work. We do not touch anything else in the repo.

After authorization, you will be authenticated by the service provider and redirected back to Shippable. You are now ready to create builds on Shippable!


-------

**Step 2** : Enable CI for repos
---------------------------------------

After logging in, you will see the Repositories on the right sidebar.  Find or search for your product in the list, and click the **Enable** button.
Now, whenever you push a commit to your GitHub repo, our webhooks will create a build for that project. Additionally, you can manually trigger builds for the enabled project, by visiting it from the **Enabled Projects** list.

-------

**Step 3** : Create YML file
----------------------------

In order for Shippable to know how to create a build for your project, you must include in it a ``shippable.yml`` file. This file must be located in the root directory of your repo.

.. note::

  This example is for a node.js project. For other languages, refer to our :ref:`language guides <langrefs>`. 

  **If you use TravisCI, we support** ``.travis.yml`` **natively, so that you can test your repos in parallel with Shippable and compare the speed and rich visualizations.**

* First, you can specify what Docker image to use. This is an optional setting and if omitted, ``shippable/minv2`` will be used (syntax is ``<docker_hub_username>/<image_name>``).
    .. code-block:: python
        
        # build image from Docker Hub (see https://registry.hub.docker.com/repos/shippableimages/)
        build_image: shippableimages/ubuntu1404_nodejs
* Next, specify the programming language of the project, and the versions of the language you'd like to create builds with. You can test against multiple versions with a single push by adding more in the versions section. 
    .. code-block:: python
        
        # language setting
        language: node_js

        # version numbers, testing against two versions of node
        node_js:
          - 0.10.25
* The ``before_install`` tag can be used to install any additional needed dependencies. Here we invoke ``npm install`` to install our Node.js app's dependencies. Even if you don't specify anything here, your minion will attempt to install dependencies for your app in an idiomatic way for the language (such as invoking rake for ruby, or pip for python)
    .. code-block:: python

        # npm install runs by default but shown here for illustrative purposes
        before_install: 
         - npm install docco
         - npm install coffee-script

* The ``script`` tag is where the magic happens. Here you will write the commands used to verify the integrity of your code. Again, if you list nothing here, your build minion will attempt to make a logical choice based on your specified language
    .. code-block:: python

        # Running npm test to run your test cases
        script: 
         - npm test

**Complete documentation of YML is available** :ref:`HERE <setup>`.

--------

**Step 4** : Setup Test Visualizations
---------------------------------------

To use Shippable's test visualization feature, your code coverage output needs to be in cobertura xml format, and test results should be in junit format. More details can be found in our :ref:`Code Samples <samplesref>`. 
This is an optional feature.


--------

**Step 5** : Run the build
---------------------------

Builds can be triggered through webhooks or manually through shippable.com. 

**Webhooks**

Our webhooks are triggered when a commit is pushed to your repo, or if a pull request is created. Webhooks are a code way to
verify that commits to your project build in a clean environment, and not just on the committer's machine.


**Manual Builds** 

After enabling the project, click the **Build this project** button to manually run a build. Instantly, it will redirect you to the build's page and the console log from your build minion starts to stream to your browser through sockets. 


--------

**Step 6** : Check output
------------------------- 
 
In addition to running builds, Shippable also provides useful visualizations for every build. 

**Console Log**:
Stdout of a build run is streamed to the browser in real-time using websockets. In addition, there are other important pieces of information like 

* build status
* duration
* GitHub changeset id
* committer info

**Artifact archive**:
If enabled, build artifacts are automatically archived for each run upon completion. To download a tarball of your build's artifacts, go to the build's page and click the **Artifacts** button. All files in the ./shippable folder at the root of the project are automatically archived. Make sure you include the **archive: true** tag in your yml file to enable the download archive button.

**Test cases**:
Test run output is streamed in real-time to the console log when the tests are executed. If you want Shippable's parser to parse test output and provide a graphical representation, you need to export a JUNIT xml of your test output to the ./shippable/testresults folder. After the build completes, our build engine will automatically parse it and the will results appear in the Tests tab (available in build's page).

**Code Coverage**:
Executing tests is only useful so far as the tests cover your code.  A variety of coverage tools like opencover, cobertura etc. provide a way to measure coverage of your tests. You can export the output of these tools to ./shippable/codecoverage and our build engine will automatically parse it. The results will appear on the Coverage tab.

Clicking the **View build history** button will take you to the project's page where you can find a complete history of your project's builds.
