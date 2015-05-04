:title: Configuring your yml 
:description: This section helps you write your shippable.yml
:keywords: getting started, questions, documentation, shippable, config, yml

.. _ymlconfig:

Configuring your yml
====================

Your shippable.yml file tells us about your project and how to run your builds and tests. This file should be at the root of your repository in order to build the repo with Shippable. To help TravisCI users quickly test our platform, we support .travis.yml natively, so you will not need a shippable.yml in addition. 

Your yml can be as minimal or as customized as necessary, depending on the project.

Build flow
----------
When we receive a build trigger through a webhook or manual run, we execute the following steps - 

1. Clone/Pull the project from Github or Bitbucket. This depends on whether the minion is in pristine state or not
2. ``cd`` into the workspace
3. Checkout the commit that is being built
4. Run the ``before_install`` commands. This is typically used to prep your minion and install/update any packages
5. Run ``install`` section. This should be used to install any project specific libraries or packages
6. Run ``before_script`` commands. Create any folders and unzip files that might be needed for testing. Some users also restore DBs, copy environment variables, etc. here
7. Run the ``script`` commands. This runs the build and all your tests
8. Run either ``after_success`` or ``after_failure`` commands, depending on the result of your build. after_success can be used to deploy to any supported cloud provider
9. Run ``after_script`` command


Build status is determined based on the outcome of the above steps. They need to return an exit code of ``0`` to be marked as success. Everything else is treated as a failure.

Any errors in ``after_script`` will not affect the status of the build.

a closer look at yml sections
-----------------------------

**build_image**

By default, all Shippable builds use the image shippable/minv2. This is a huge image and comes pre-installed with all popular language versions, along with all supported tools and services.  If you want to use the default image, you do not need the ``build_image`` tag in your yml.

The recommended approach is to use one of our `language specific images <http://docs.shippable.com/en/latest/custom_images.html#language-specific-images>`_ 

``build_image: shippableimages/ubuntu1204_go

Please note that language specific images do not have any tools and services pre-installed, so you will need to install these in your yml.

**language**

You should specify the programming language and versions of the language you'd like to build against. You can test against multiple versions with a single commit by adding more in the versions section. 

    .. code-block:: python
        
        # language setting
        language: node_js

        # version numbers, testing against two versions of node
        node_js:
          - 0.10.25
          - 0.11

**before_install**

The ``before_install`` section is used to prep your build minion with any packages or dependencies that are not already pre-installed. 

For example, if you require docco and coffee_script for your Node.js app, you would do the following -

    .. code-block:: python

        # npm install runs by default but shown here for illustrative purposes
        before_install: 
         - npm install docco
         - npm install coffee-script


If nothing is specified in this section, we will attempt to install dependencies for your app in an idiomatic way for the language (such as invoking rake for ruby, or pip for python)

**install**

The install section lets you install packages and dependencies that are specific to your project. This is useful if your project uses non standard tools. 

    .. code-block:: python

       #install java project dependencies with ant
        install: ant deps

**script**

The ``script`` section is where the magic happens. In this section, you can specify the main build commands for your project.
Again, if you list nothing here, your build minion will attempt to make a logical choice based on your specified language.

    .. code-block:: python

        # Running npm test to run your test cases
        script: 
         - npm test

You can run any script file as part of your configuration, as long as it has a valid shebang command and the right ``chmod`` permissions. 

.. code-block:: python
        
        # script file 
        script: ./minions/do_something.sh 


If you want to prevent shippable from using the default build command you can add following:

.. code-block:: python
        
        # script file 
        script: 
        - true #or any custom command

**after_success or after_failure**

These sections are used to specify commands to be called after the build succeeds or fails. 
For example, for a Java project using Cobertura, this section can be used to clean up files created during instrumentation. 

.. code-block:: python

   after_success:
      - mvn clean cobertura:cobertura

Commonly, ``after_success`` section is also used to add deployment scripts 

.. code-block:: python

    after_success:
        - test -f ~/.ssh/id_rsa.heroku || ssh-keygen -y -f ~/.ssh/id_rsa > ~/.ssh/id_rsa.heroku && heroku keys:add ~/.ssh/id_rsa.heroku
        - git remote -v | grep ^heroku || heroku git:remote --ssh-git --app $APP_NAME
        - git push -f heroku master

**after_script**

This is the last user defined section to be executed, and can be used to perform tasks after the build and tests are complete, like generating a coverage report -

.. code-block:: python

   # Tell istanbul to generate a coverage report
    after_script:
        - ./node_modules/.bin/istanbul cover grunt -- -u tdd
        - ./node_modules/.bin/istanbul report cobertura --dir  shippable/codecoverage/

useful yml tags
---------------

**command collections**
``shippable.yml`` supports collections under each tag. This is nothing more than YML functionality and we will run it one command at a time.

.. code-block:: python
        
  # collection scripts 
  script: 
   - ./minions/do_something.sh 
   - ./minions/do_something_else.sh 

In the example above, our minions will run ``./minions/do_something.sh`` and then run ``./minions/do_something-else.sh``. The only requirement is that all of these operations return a ``0`` exit code. Else the build will fail.

shippable_retry
.................

Sometimes npm install may fail due to the intermittent network issues and affects your build execution. To avoid this, **shippable_retry** function will try to install the command again. It will check the return code of a command and if it is non-zero, then it will re-try to install up to three times.

**shippable_retry** functionality is available for all default installation commands and it will re-try to install on failure. You can also use this functionality for any custom installation from external resources. For example:

.. code-block:: python
  
    before_install:
        - shippable_retry sudo apt-get update
        - shippable_retry sudo apt-get install something




git submodules
..............
Shippable supports git submodules. This is a cool functionality of breaking your projects down into manageable chunks. We automatically initialize the ``.gitmodules`` file in the root of the repo. 

.. note::

  If you are using private repos, add the deploy keys so that our minion ssh keys are allowed to pull from the repo. This can be done via shippable.com

If its your own public repos then do this

.. code-block:: python
        
  # for public modules use
  git://github.com/someuser/somelibrary.git

  # for private modules use
  git@github.com:someuser/somelibrary.git

If you would like to turn submodules off completely -

.. code-block:: python
        
  # for public modules use
  git:
   submodules: false


  
common environment variables
.............................

The following environment variables are available for every build. You can use these in your scripts if required -

- BRANCH : Name of branch being built

- BASE_BRANCH : Name of the target branch into which the pull request changes will be merged 

- BUILD_NUMBER : Build number for current build

- BUILD_URL : Direct URL link to the build output

- CI : true

- CONTINUOUS_INTEGRATION : true  

- COMMIT : Commit id that is being built and tested

- COMPARE_URL : A link to GitHub/BitBucket's comparision view for the push
 
- DEBIAN_FRONTEND : noninteractive

- HEAD_BRANCH: Name of the most recently committed branch

- JOB_ID : id of job in Shippable

- LANG : en_US.UTF-8

- LAST_SUCCESSFUL_BUILD_TIMESTAMP : Timestamp of the last successful build in seconds. This will be set to **false** for the first build or for the build with no prior successful builds 

- LC_ALL : en_US.UTF-8

- LC_CTYPE : en_US.UTF-8

- MERB_ENV : test

- PATH : $HOME/bin:$PATH

- PULL_REQUEST : Pull request number if the job is a pull request. If not, this will be set to **false**

- RACK_ENV : test

- RAILS_ENV : test

- REPO_NAME : Name of the repository currently being built

- REPOSITORY_URL : URL of your Github or Bitbucket repository

- SERVICE_SKIP : false

- SHIPPABLE : true

- SHIPPABLE_ARCHIVE : true

- SHIPPABLE_BUILD_ID : id of build in Shippable 

- SHIPPABLE_MYSQL_BINARY : "/usr/bin/mysqld_safe"

- SHIPPABLE_MYSQL_CMD : "$SHIPPABLE_MYSQL_BINARY"

- SHIPPABLE_POSTGRES_VERSION : "9.2"

- SHIPPABLE_POSTGRES_BINARY : "/usr/lib/postgresql/$SHIPPABLE_POSTGRES_VERSION/bin/postgres" 

- SHIPPABLE_POSTGRES_CMD : "sudo -u postgres $SHIPPABLE_POSTGRES_BINARY -c \"config_file=/etc/postgresql/$SHIPPABLE_POSTGRES_VERSION/main/postgresql.conf\""

- SHIPPABLE_VE_DIR : "$HOME/build_ve/python/2.7"

- USER : shippable


user specified environment variables
.....................................

You can set your own environment variables in the yml. Every statement of this command will trigger a separate build with that specific version of the environment variables. 

.. code-block:: python
        
  # environment variable
  env:
   - FOO=foo BAR=bar
   - FOO=bar BAR=foo


.. note::

  Env variables can create an exponential number of builds when combined with ``jdk`` & ``rvm , node_js etc.`` i.e. it is multiplicative

In this setting **4 individual builds** are triggered in a build group

.. code-block:: python
        
  # npm builds
  node_js:
    - 0.10.24
    - 0.8.14
  env:
    - FOO=foo BAR=bar
    - FOO=bar BAR=foo

.. _secure_env_variables:

Secure environment variables
.............................

Shippable allows you to encrypt the environment variable definitions and keep your configurations private using **secure** tag. Go to the org dashboard  or individual dashboard page from where you have enabled your project and click on **ENCRYPT ENV VARIABLE** button on the top right corner of the page. Enter the env variable and its value in the text box as shown below. 

.. code-block:: python

    name=abc

Click on the encrypt button and copy the encrypted output string and add it to your yml file as shown below:


.. code-block:: python
   
   env:
     secure: <encrypted output>


To encrypt multiple environment variables and use them as part of a single build, enter the environment variable definitions in the text box as shown below 

.. code-block:: python

  name1="abc" name2="xyz"    

This will give you a single encrypted output that you can embed in your yml file.


You can also combine encrypted output and clear text environments using **global** tag. 

.. code-block:: python
 
   env:
     global:
       - FOO="bar"
       - secure: <encrypted output>


To encrypt multiple environment variables separately, configure your yml file as shown below: 

.. code-block:: python
  
  env:
    global:
      #encrypted output of first env variable
      - secure: <encrypted output> 
      #encrypted output of second env variable
      - secure: <encrypted output>
    matrix:
      #encrypted output of third env variable
      - secure: <encrypted output>

.. note::

   Due to the security risk of exposing your secure variables, we do not decrypt secure variables for pull request from the forks of public projects. Secure variable decryption is limited to the pull request triggered from the branches on the same repository. And the decrypted secured variables are also not displayed in the script tab for security reasons. 	



include & exclude branches
..........................

By default, Shippable builds all branches for enabled repositories as long as they have a shippable.yml at the root. 

You can change this build only specific branches using the include and exclude sections in your yml. The specific branch that is being included or excluded needs to have this configuration, and not just the master branch. 

This is because Shippable works as follows - we get a webhook for an enabled repository letting us know something has changed in a specific branch. We read the shippable.yml from that branch and then trigger a build based on that. So if your shippable.yml in the develop branch does not contain the exclude section, we will trigger a build irrespective of what's in the yml in master branch.

Here is a sample of the include/exclude config - 

.. code-block:: python

  # exclude
  branches:
    except:
      - test1
      - experiment2

  # include
  branches:
    only:
      - stage
      - prod


build matrix
............

This is another powerful feature that Shippable has to offer. You can trigger multiple different test passes for a single code push. You might want to test against different versions of ruby, or different aspect ratios for your Selenium tests or best yet, just different jdk versions. You can do it all with Shippable's matrix build mechanism.

.. code-block:: python

  rvm:
    - 1.8.7 # (current default)
    - 1.9.2
    - 1.9.3
    - rbx
    - jruby
    - ruby-head
    - ree
  gemfile:
    - gemfiles/Gemfile.rails-2.3.x
    - gemfiles/Gemfile.rails-3.0.x
    - gemfiles/Gemfile.rails-3.1.x
    - gemfiles/Gemfile.rails-edge
  env:
    - ISOLATED=true
    - ISOLATED=false

The above example will fire 36 different builds for each push. Whoa! Need more minions?
 

**exclude**

It is also possible to exclude a specific version using exclude tag. Configure your yml file as shown below to exclude a specific version.

.. code-block:: python

   matrix:
     exclude:
       - rvm: 1.9.2
        


**include**

You can also configure your yml file to include entries into the matrix with include tag.

.. code-block:: python

   matrix:
     include:
       - rvm: 2.0.0
         gemfile: gemfiles/Gemfile.rails-3.0.x
         env: ISOLATED=false


**allow-failures**

Allowed failures are items in your build matrix that are allowed to fail without causing the entire build to be shown as failed. You can define allowed failures in the build matrix as follows:

.. code-block:: python

  matrix:
    allow_failures:
      - rvm: 1.9.3



