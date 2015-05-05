:title: Getting Started 
:description: Basic getting started section
:keywords: getting started, questions, documentation, shippable

.. _getstarted:

Getting Started
===============


**Step 1** : Sign Up
--------------------

You can sign in to Shippable using a GitHub or Bitbucket account. We use OAuth authentication, so you do not need to create a separate account on our platform. 

To sign in, visit the `Shippable website <https://www.shippable.com>`_, click **Login**, and choose between Github or Bitbucket auth.

After entering your credentials, you will be prompted to give Shippable access to your repos. GitHub and Bitbucker auth behave a little differently as follows -

**GitHub**- By default, we will only ask for access to public repos. If you want to use Shippable to build your private repos, you will need to authorize us for private repositories. To do that, click on Private Repos off icon at the top right of your dashboard and go through the GitHub auth flow. The 'Private Repos' icon should show 'ON' when you return to the dashboard.

**Bitbucket**- The Bitbucket API does not have public/private granularity, so we ask for access to all repos on Bitbucket by default.

.. note::

    We realize that most people do not want to give write access to their repo. However, we need write permissions to add deploy keys to your repos for our webhooks to work. We do not touch anything else in the repo.

You are now ready to build and test with Shippable!

-------

**Step 2** : Enable CI for repos
---------------------------------------

After logging in, you will see 'Repositories' on the right sidebar of your dashboard.  Find or search for your repository in the list, and click the **Enable** button.
Now, whenever you push a commit to your GitHub repo, Shippable will build that project as long as you have a shippable.yml (or .travis.yml) at the root of your repository. 

-------

**Step 3** : Create YML file
----------------------------

We require a shippable.yml file at the root of the repository you want to build with Shippable. This yml file is the config for your build.

.. note::

  **If you use TravisCI, we support** ``.travis.yml`` **natively, so that you can test your repos in parallel with Shippable and compare the speed and rich visualizations.**


Your yml needs a couple of entries at the very minumum - the language and the version(s) of the language you want to test against. 

.. code-block:: python
   
    # language setting
    language: node_js

    # version numbers, testing against two versions of node
    node_js:
        - 0.10.25
        - 0.11

The rest of the yml is composed of standard sections - ``before_install``, ``install``, ``before_script``, ``script``, ``after_success`` or ``after_failure``, and ``after_script``.

If you just have the language and language version in your yml, we will attempt to 'guess' at the other settings and build your project. However, in most cases, you will need to specify additional configuration in the yml.

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
Test run output is streamed in real-time to the console log when the tests are executed. If you want Shippable's parser to parse test output and provide a graphical representation, you need to export a JUNIT xml of your test output to the ./shippable/testresults folder. After the build completes, our build engine will automatically parse it and the results will appear in the Tests tab (available in build's page).

**Code Coverage**:
Executing tests is only useful so far as the tests cover your code.  A variety of coverage tools like opencover, cobertura etc. provide a way to measure coverage of your tests. You can export the output of these tools to ./shippable/codecoverage and our build engine will automatically parse it. The results will appear on the Coverage tab.

Clicking the **View build history** button will take you to the project's page where you can find a complete history of your project's builds.
