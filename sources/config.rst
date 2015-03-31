:title: Shippable Configuration
:description: Complete guide to how Shippable is setup
:keywords: shippable, signup, login, registration
	
.. _setup:

A bit deeper
============

Build Minions and Build Configuration are two things that you should care the most about when using Shippable. The sections below talk about these in greater detail.


**Minions**
-----------

Minions are Docker based containers that run your builds and tests. You can think of them as Build VMs. Each minion runs one build at a time, so the more minions you have, the more concurrency you will get.  

Minions are automatically provisioned whenever a build is triggered and it will get deleted after the build finishes execution. We will automatically add additional minions, as appropriate, based on your subscription plan.

Each minion starts from a base image and can be customized by specifying ``before_install`` scripts in the YML file. A minion can be configured to run any package, library, or service that your application requires. There are some preinstalled tools and services that you can use to customize your minions even further. 

Operating Systems
.................

All our Linux minions start from a vanilla base image from the Docker registry. In your YML file you can use the ``build_image`` option to specify the image you want to use. We support all images as a starting point for your minion. Minions can be further customized by using the ``before_install`` and ``install`` tags in ``shippable.yml`` that is in the root of your code repository.

(Coming soon) Our Windows minions are based on AWS AMI for Windows 2012.



Common Tools
............

A set of common tools are available on all minions. The following is a list of available tools -

- Latest release of Git repository
- apt installer
- Networking tools  
  
  - curl
  - wget
  - OpenSSL

- At least 1 version of (check out the language documentation for specific versions)
  
  - Ruby
  - Node
  - Python 
  - OpenJDK
  - PHP
  - Go
  - Clojure

- Services
  
  - MySQL
  - Postgre
  - SQLite
  - MongoDB
  - Redis
  - ElasticSearch
  - Selenium Server
  - Neo4j
  - Cassandra
  - CouchDB
  - RethinkDB

- Headless browser testing tools

  - xfvb
  - PhantomJS

- Libraries

  - ImageMagik
  - OpenSSL

- JDK Versions

  - Oracle JDK 7u6 (oraclejdk7)
  - OpenJDK 7 (alias: openjdk7)
  - OpenJDK 6 (openjdk6)
  - Oracle JDK 8 EA (oraclejdk8)

- Build Tools

  - Maven 3
  - Gradle 1.9
  - Make
  - SBT 0.12.1

- Preinstalled PIP Packages

  - mock
  - nosetests
  - py.test

- Gems

  - Bundler
  - Rake
 
- Addons
  
  - Firefox
  - Custom hostname
  - PostgreSQL

----------

**Configuration**
------------------

This section is generic to all build environments and all languages. If you are looking for language specific tags, please refer to our language guides for more information.

``shippable.yml``
.................

We believe that developers need complete control of their build configuration. We also realize that most developers don't want to log into a UI to make changes every single time. 

While mulling over the best way to give developers control without asking them to go through our UI, we came across ``.travis.yml``, an open source initiative that created the basic framework for this very problem. Following the same paradigm, we ask you to have ``shippable.yml`` in the root of the repository you want to build. The structure of shippable.yml closely mimics travis since we see no reason to reinvent the wheel. We do have additional tags for added functionality, and these will become more numerous as we evolve Shippable. 

Since shippable.yml is a superset of ``.travis.yml`` , we support ``.travis.yml`` natively as well. So if you have a travis.yml in the root of your repo, we will read the config and set up your CI.

At a minimum, Shippable needs to have your language and build version specified in the yml. We will then default to the most common commands.

Build Flow
..........

When we receive a build trigger through a webhook or manual run, we execute the following steps - 

1. Clone/Pull the project from Github or Bitbucket. This depends on whether the minion is in pristine state or not
2. ``cd`` into the workspace
3. Checkout the commit that is getting built
4. Run the ``before_install`` section. This is typically used to prep your minion and update any packages
5. Run ``install`` section to install any project specific libraries or packages
6. Run ``before_script`` section to create any folders and unzip files that might be needed for testing. Some users also restore DBs etc. here
7. Run the ``script`` command which runs build and all your tests
8. Run ``after_script`` command
9. Run either ``after_success`` or ``after_failure`` commands


Build status will be determined based on the outcome of the above steps. They need to return an exit code of ``0`` to be marked as success. Everything else is treated as a failure.


----------

**Other useful configs**
------------------------

Shippable uses Docker containers to provide you with isolation and a dedicated build environment. Our command sessions are not sticky throughout the build, but they are sticky within the same section of the build. For e.g. ``cd`` is sticky within the ``before_script`` tag of ``shippable.yml``

script
......
You can run any script file as part of your configuration, as long as it has a valid shebang command and the right ``chmod`` permissions. 

.. code-block:: python
        
        # script file 
        script: ./minions/do_something.sh 


If you want to prevent shippable from using the default build command you can add following:

.. code-block:: python
        
        # script file 
        script: 
        - true #or any custom command


command collections
...................
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



----------

**Services**
-----------------
Shippable offers a host of pre-installed services to make it easy to run your builds. In addition to these you can install other services also by using the ``install`` tag of ``shippable.yml``. 

All the services are turned off by default and can be turned on by using the ``services:`` tag.

MongoDB
.......

.. code-block:: bash
  
  # Mongo binds to 127.0.0.1 by default
  services:
   - mongodb

Sample PHP code using `mongodb <https://github.com/shippableSamples/sample_php_mongo>`_ .


MySQL
.....

.. code-block:: bash
  
  # MySQL binds to 127.0.0.1 by default and is started on boot. Default username is shippable with no password
  # Create a DB as part of before script to use it

  before_script:
      - mysql -e 'create database myapp_test;'
                                 
Sample javascript code using `mysql <https://github.com/shippableSamples/sample_node_mysql>`_.


SQLite3
.......

SQLite is a software library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine. So you can use SQLite, if you do not want to test your code behaviour with other databases.

Sample python code using `SQLite <https://github.com/shippableSamples/sample_python_sqllite>`_.


Elastic Search
..............

.. code-block:: bash

  # elastic search is on default port 9200
  services:
      - elasticsearch

Sample python code using `Elastic Search <https://github.com/shippableSamples/sample_python_elasticsearch>`_.

Memcached
..........

.. code-block:: bash

  # memcached runs on default port 11211
  services:
      - memcached

Sample python code using `Memcached <https://github.com/shippableSamples/sample_python_memcache>`_ .


Redis
.....

.. code-block:: bash

  # redis runs on default port 6379
  services:
      - redis


Sample python code using `Redis <https://github.com/shippableSamples/sample_python_redis>`_.


Neo4j
.....

.. code-block:: bash
 
 #neo4j runs on default port 7474
 services:
  - neo4j

Sample javascript code using `Neo4j <https://github.com/shippableSamples/sample_node_neo4j>`_ .

Cassandra
..........

.. code-block:: bash
 
 # cassandra binds to the default localhost 127.0.0.1 and is not started on boot. 
 services:
   - cassandra

Sample ruby code using `Cassandra <https://github.com/shippableSamples/sample_ruby_cassandra>`_ .

CouchDB
.........

.. code-block:: bash

 # couchdb binds to the default localhost 127.0.0.1 and runs on default port 5984. It is not started on boot.
 services:
   - couchdb

Sample ruby code using `CouchDB <https://github.com/shippableSamples/sample-ruby-couchdb>`_ .

RethinkDB
...........

.. code-block:: bash

 # rethinkdb binds to the default localhost 127.0.0.1 and is not started on boot.
 services:
   - rethinkdb

Sample javascript code using `RethinkDB <https://github.com/shippableSamples/sample-node-rethinkdb>`_.
 
RabbitMQ
.........

.. code-block:: bash

  # rabbitmq binds to 127.0.0.1 and is not started on boot. Default vhost "/", username "guest" and password "guest" can be used.
  services:
    - rabbitmq

Sample python code using `RabbitMQ <https://github.com/shippableSamples/sample_python_rabbitmq>`_ .


--------

**Addons**
----------

firefox
..........

We support different firefox versions like "18.0", "19.0", "20.0", "21.0", "22.0", "23.0", "24.0", "25.0", "26.0", "27.0", "28.0", "29.0". To select a specific firefox version, add the following to your shippable.yml file.

.. code-block:: python

	addons:
  	   firefox: "21.0"

custom host name
..................

You can also set up custom hostnames using the **hosts** addons. To set up the hostnames in /etc/hosts file, add the following to your shippable.yml file.
   
.. code-block:: python

        addons:
           hosts: 
    	    - google.com
            - asdf.com

PostgreSQL
...........

.. code-block:: bash

  # Postgre binds to 127.0.0.1 by default and is started on boot. Default username is "postgres" with no password
  # Create a DB as part of before script to use it

  before_script:
    - psql -c 'create database myapp_test;' -U postgres

Sample java code using `PostgreSQL <https://github.com/shippableSamples/sample_java_postgres>`_.

We support PostgreSQL 9.1, 9.2 and 9.3 versions and by default, version 9.2 is installed on our minions. Configure your yml file using **PostgreSQL** addons to select different versions. Add the following to your yml file to select the version 9.3.


.. code-block:: python

          addons:
           postgresql : "9.3"
  
PostGIS 2.1 packages are pre-installed in our minions along with the PostgreSQL versions 9.1, 9.2 and 9.3.


Selenium
...........


Selenium is not started on boot. You will have to enable it using **services** tag and start xvfb (X Virtual Framebuffer) on display port 99.0, so that all your test suites will run on the server without a display. Configure your yml file as shown below to start selenium server on firefox.

.. code-block:: bash
   
     addons:
       firefox: "23.0"

     services:
       - selenium

     before_script:
       - "export DISPLAY=:99.0"
       - "/etc/init.d/xvfb start"
     
Selenium **2.40** is started by default. You can also select a different version of selenium using **addons** tag. The following versions are supported:
 
- 2.39
- 2.40
- 2.41
- 2.42
- 2.43
- 2.44 

Choose the required version and add it to your shippable.yml file as shown below  

.. code-block:: bash

          addons:
            selenium: "2.43"

This will download the required version. You will have to include **services** tag in your yml file to start the selenium server using the downloaded version. Configure your yml file as shown below to start selenium server **2.43** on firefox.
      
.. code-block:: bash
 
    #specify required selenium and firefox version
    addons:
      selenium: "2.43"
      firefox: "27.0"
     
    #start the selenium server  
    services:
      - selenium

    before_script:
      - "export DISPLAY=:99.0"
      - "/etc/init.d/xvfb start"
  


Sample javascript code using `Selenium <https://github.com/shippableSamples/sample_node_selenium>`_ .



----------

**Test and Code Coverage visualization**
----------------------------------------
Test results
............
To set up test result visualization for a repository.

* Output test results to shippable/testresults folder. 
* Make sure test results are in junit format.

For example, here is the .yml file for a Python repo -

.. code-block:: bash

  before_script: mkdir -p shippable/testresults
  script:
    - nosetests python/sample.py --with-xunit --xunit-file=shippable/testresults/nosetests.xml

Examples for other languages can be found in our :ref:`Code Samples <samplesref>` .

Code coverage
.............
To set up code coverage result visualization for a repository.

* Output code coverage output to shippable/codecoverage folder. 
* Make sure code coverage output is in cobertura xml format.

For example, here is the .yml file for a Python repo -

.. code-block:: bash

  before_script: mkdir -p shippable/codecoverage
  script:
    - coverage run --branch python/sample.py
    - coverage xml -o shippable/codecoverage/coverage.xml python/sample.py

Examples for other languages can be found in our :ref:`Code Samples <samplesref>`.


-------------

**Notifications**
-----------------
Shippable primarily supports email and irc notifications and these can can be configured through the yml. To send Slack notifications, please check out our `blog post <http://blog.shippable.com/devops-chat-a-simple-way-to-use-slack-notifications-with-shippable>`_.  To send HipChat notifications, check out our `sample project for hipchat notifications <https://github.com/shippableSamples/sample-hipchat-notifications>`_.

By default, we send email notifications to the last committer when a build fails, or the status changes from failed to passed.

You can change the default settings for email notifications by configuring the notifications section of your yml. You can specify the email address(es) where you want to receive notification as well as the criteria for when you want notifications to be sent. 

Email notification
..................


To send notifications to specific email addresses, replace the sample email addresses below with the recipients' email ids in your ``shippable.yml`` file.

.. code-block:: bash

  notifications:
      email:
          - exampleone@org.com
          - exampletwo@org.com


You can also specify when you want to get notified by setting the values for on_success and on_failure keys to change|always|never. Change means you want to be notified only when the build status changes on the given branch. Always and never mean you want to be notified always or never respectively.


.. code-block:: bash
 
  notifications:
       email:
           recipients:
               - exampleone@org.com
               - exampletwo@org.com
           on_success: change
           on_failure: always


If you do not want to get notified for any reason, you can configure email notifications to false.

.. code-block:: bash

  notifications:
     email: false


IRC notification
..................

You can also configure yml file to send build notifications to your IRC channels. 

- To specify single channel

.. code-block:: bash

   notifications:
      irc:  "chat.freenode.net#channel1"

- You can also specify multiple server channels in yml file. The following formats are supported: 

.. code-block:: bash

   notifications:
     irc: 
       - "chat.freenode.net#channel1"
       - "chat.freenode.net#channel2"
       - "server1#channel3"


  
.. code-block:: bash

  notifications:
    irc:
     channels: 
       - "chat.freenode.net#channel1"
       - "chat.freenode.net#channel2"
       - "server1#channel3"

- By default, We will always send build notifications to the mentioned channels in yml. **on_success** and **on_failure**  are not yet configurable. 

- IRC notifications are turned off by default for pull request builds. However, you can change the default settings by adding **pull_requests: true** tag in your yml as shown below.

.. code-block:: bash

  notifications:
    irc:
     pull_requests: true
     channels:
       - "chat.freenode.net#channel1"
 

----------

**Pull Request**
------------------


Shippable will integrate with github to show your pull request status on CI. Whenever a pull request is opened for your repo, we will run the build for the respective pull request and notify you about the status. You can decide whether to merge the request or not, based on the status shown. If you accept the pull request, Shippable will run one more build for the merged repo and will send email notifications for the merged repo.
To rerun a pull request build, go to your project's page -> Pull Request tab and then click on the **Build this Pull Request** button.
 
--------

**Permissions**
------------------

We will automatically add your collaborators when you login to shippable and it will be updated in the user-interface. 


There are two types of roles that users can have -

**Owner :** 
Owner is the highest role. This role permits users to create, run and delete a project. 


**Collaborator :** 
Collaborator can run or manage projects that are already setup. They have full visibility into the project and can trigger the build.



--------

**Build Termination**
-----------------------

Build will be forcefully terminated in the following scenarios:

* If a command hangs for a long time or there hasn't been any log output   
* If the build is still executing after 60 minutes 

and the status of the build will be updated as **timeout**.

.. note::
  
     Build termination time depends on the pricing plan. If it is a **Startup** plan, then the build will be terminated after 90 minutes. 	
 


 
--------

**Skipping a build**
-----------------------

Any changes to your source code will trigger a build automatically on Shippable. So if you do not want to run build for a particular commit, then add **[ci skip]** or **[skip ci]** to your commit message. 

Our webhook processor will look for the string  **[ci skip]** or **[skip ci]** in the commit message and if it exists, then that particular webhook build will not be executed.

--------

**Using Shippable with Gitlab or other types of source control**
----------------------------------------------------------------

At the moment, Shippable supports repositories hosted either on GitHub or Bitbucket.
However, your development setup may involve using a different provider or even hosting the repository server on your own.
In both cases, the easiest way to make your code available to Shippable is to set up a mirror of your repository with either of the supported services. 

As `GitLab <https://about.gitlab.com/>`_ is a very popular choice among organizations managing their own repositories,
the instructions below outline how to set up a mirror of a repository hosted on GitLab Community Edition 7.2.1.
Other self-hosted solutions can be integrated in a very similar manner, the only differences being the locations of the files.
If you experience any problems setting up a mirror using a different technology, please do not hesitate to reach out to us.
Please also note that this method can be applied to mirror repositories using different VCS than Git or Mercurial, if only an extension to push the changes to Git is available.
Hence, it allows using VCS of your choice (such as Perforce or SVN) with Shippable.

First step of setting up a mirror is to create a target repository on either GitHub or Bitbucket.
Please note that in the case of Bitbucket, you can create unlimited number of private repositories for free, granted that no more than 5 users will have access to them.
It makes it especially appealing to host mirrors then, as you only need to associate two users (the account you use with Shippable and an extra account for your repository server) with all the mirrors.

Please write down the Git url of your newly created mirror. For the sake of convenience, we will use the following urls throughout this guide:

* GitHub: ``git@github.com:Shippable/shippable-mirror-test.git``
* Bitbucket: ``git@bitbucket.org:Shippable/shippable-mirror-test.git``

Next, you need to grant write access for your repository server to the mirror.

Granting access to a GitHub mirror
..................................

In case of GitHub, it can be done by `adding a deployment key <https://developer.github.com/guides/managing-deploy-keys/#deploy-keys>`_ to the repository. 
GitHub requires the deployment keys to be unique, so we need to create a dedicated SSH key for every repository.

Switch to the ``git`` user on your GitLab server and create the key, using a filename that clearly associates the key with the repository.
Leave the passphrase empty:

.. code-block:: bash

  # su - git
  $ bash
  $ pwd
  /var/opt/gitlab
  $ ssh-keygen -f .ssh/shippable-mirror_key

Next, take the contents of the ``.ssh/shippable-mirror_key.pub`` file and add it as a deploy key in the GitHub settings panel for your mirror repository.
To ensure that the right key gets picked when ``git`` establishes the connection with the mirror, we will add a special host entry in the SSH config.
Open ``/var/opt/gitlab/.ssh/config`` file (create it if it doesn't exist) with your favorite editor and add the following section:

.. code-block:: bash

  Host shippable-mirror
  IdentityFile /var/opt/gitlab/.ssh/shippable-mirror_key
  HostName github.com  
  User git

Now, connecting to ``shippable-mirror`` host (replace this alias with your repository name) will result in establishing a connection with ``github.com`` as user ``git``,
but using our dedicated deployment key. Test it by issuing the following command (still as ``git`` user on your GitLab server):

.. code-block:: bash

  $ ssh shippable-mirror
  Warning: Permanently added the RSA host key for IP address '192.30.252.129' to the list of known hosts.
  PTY allocation request failed on channel 0
  Hi Shippable/shippable-mirror-test! You've successfully authenticated, but GitHub does not provide shell access.
  Connection to github.com closed.

If you see a message like this above, you have successfully set the deployment key up.

Granting access to a Bitbucket mirror
.....................................

In case of Bitbucket, you need to create an SSH key and associate it with a user account.
As it is not advisable to deploy to a remote server the key that grants access to your private account,
we recommended creating a separate user on Bitbucket.org for authenticating your GitLab server.

You can create one key for ``git`` user on your GitLab server and use it for all the services, but for security reasons you may create
a separate key for Bitbucket.  Switch to the ``git`` user on your GitLab server and create the key, leaving the passphrase empty:

.. code-block:: bash

  # su - git
  $ bash
  $ pwd
  /var/opt/gitlab
  $ ssh-keygen -f .ssh/bitbucket_key

Next, take the contents of the ``.ssh/bitbucket_key.pub`` file and 
`add the key for it in the account management panel <https://confluence.atlassian.com/display/BITBUCKET/Add+an+SSH+key+to+an+account>`_.
To ensure that the right key gets picked when ``git`` establishes the connection with the mirror, we will add a special host entry in the SSH config.
Open ``/var/opt/gitlab/.ssh/config`` file (create it if it doesn't exist) with your favorite editor and add the following section:

.. code-block:: bash

  Host bitbucket
  IdentityFile /var/opt/gitlab/.ssh/bitbucket_key
  HostName bitbucket.org
  User git

Now, connecting to ``bitbucket`` host will result in establishing a connection with ``bitbucket.org`` as user ``git``,
but using our dedicated deployment key. Test it by issuing the following command (still as ``git`` user on your GitLab server):

.. code-block:: bash

  $ ssh bitbucket
  Warning: Permanently added the RSA host key for IP address '131.103.20.168' to the list of known hosts.
  PTY allocation request failed on channel 0
  logged in as Shippable.

  You can use git or hg to connect to Bitbucket. Shell access is disabled.
  Connection to bitbucket.org closed.

If you see a message like this above, you have successfully set the deployment key up.

Setting a git hook
..................

The next step is to add mirror as a remote to the repository on your GitLab server.
Still as ``git`` user, go to the directory that contains the repository and issue the following command:

.. code-block:: bash

  $ cd /var/opt/gitlab/git-data/repositories/<GitLab namespace for the repo>/<repo name>.git
  # for GitHub
  $ git remote add mirror --mirror=push shippable-mirror:Shippable/shippable-mirror-test.git
  # for Bitbucket 
  $ git remote add mirror --mirror=push bitbucket:Shippable/shippable-mirror-test.git        

Next, add the ``post-receive`` hook in the same directory (you can read more about ``git`` hooks `here <http://git-scm.com/docs/githooks.html>`_).

.. code-block:: bash

  $ echo "exec git push --quiet mirror &" >> hooks/post-receive
  $ chmod 755 hooks/post-receive

You are now all set! After you push new changes to the GitLab, they whole repository will be automatically mirrored to either GitHub or Bitbucket.
If any errors occur, they should be visible in the output of your local ``git push`` command.

Mirroring Mercurial repository
..............................

At the very moment, Shippable supports only Git repositories.
This will likely change in the near future, but in the meantime, the only way to use a Mercurial repository with Shippable is to set up a Git mirror.

A bridge between Mercurial and Git exists in form of `Hg-Git Mercurial plugin <http://hg-git.github.io/>`_.
This plugin needs to be installed on the machine from which pushes to Git repositories will be performed
(i.e. developer workstation or server that hosts Mercurial repository).

Installation of the plugin is simple:

1. Install the plugin package with ``easy_install hg-git`` or ``pip install hg-git``.
2. Add the plugin to ``.hgrc`` file (either ``~/.hgrc`` to enable the plugin globally or ``<repository>/.hg/hgrc`` to switch it on for individual repository):

.. code-block:: bash

  [extensions]
  hggit = 

Then, add the remote endpoint for the Git repository to the list of paths in the ``<repository>/.hg/hgrc`` file.
The url needs to have ``git+ssh`` protocol in order to be picked up by ``hg-git`` plugin. 

.. code-block:: bash

  [paths]
  default = ssh://hg@bitbucket.org/Shippable/hg-mirror-test
  git = git+ssh://git@bitbucket.org/Shippable/hg-mirror-test-git.git

Then, it should be possible to send the changesets to the Git repository with the following command
(see the sections above for details how to configure SSH authentication for GitHub and Bitbucket):

.. code-block:: bash

  $ hg push git
  pushing to git+ssh://git@bitbucket.org/Shippable/hg-mirror-test-git.git
  ["git-receive-pack '/Shippable/hg-mirror-test-git.git'"]
  searching for changes
  adding objects
  added 1 commits with 1 trees and 0 blobs
  adding reference refs/heads/master

Next, we can create Mercurial hooks to perform the push automatically.
For example, when configuring the mirror on Mercurial repository server, the best idea is to use
``changegroup`` hook that gets invoked after every group of changesets that arrive at the repository.

Add the following section to ``<repository>/.hg/hgrc`` file:

.. code-block:: bash

  [hooks]
  changegroup = hg bookmark -f -r tip master && hg push -r tip git

The first command in the snippet above moves the ``master`` bookmark that
is used by ``hg-git`` to track the remote ``master`` branch from the Git repository.

There are a few caveats related to branch handling differences between Mercurial and Git.
Git branches are most akin to Mercurial bookmarks and ``hg-git`` handles them this way.
No good equivalent of Mercurial branches exist in Git, so these will not be synchronized to the Git repository,
unless an extra bookmark is created to track the branch. Please refer to ``hg-git`` documentation for more details.

One particular problem connected to this issue is that Mercurial bookmarks are bound to the local repository and
are not pushed. This means that the bookmarks are invisible to 'central' repository that has the hook in place (and sends the changes to Git).
The only solution here is to clone (i.e. fork) Mercurial repositories and keep separate Git mirrors for them, instead of creating branches.

---------

**Docker hub**
---------------

Shippable allows you to push the containers to docker registry after a successful build. To avail this option, you will have to enable the Docker hub from shippable account first. Follow the steps below to enable and push the container to docker registry.

1. Select the source code hosted account from the dashboard. It will redirect you to the selected account's dashboard page.
2. Click on the **Docker Hub** button on the top and then enter the docker hub credentials.
3. Configure your yml file as shown below to push the container.


.. code-block:: bash

    commit_container: username/sample_project


Here you should use the same user name that you used to sign up on docker hub with. 


-------

**Build Badge**
-------------------

Badges will display the status of your default branch. You can find the build badges on the project's page. Click on the **Badge** button and copy the markdown to your README file to display the status of most recent build on your Github or Bitbucket repo page.


--------

**Caching minions**
-------------------------

Shippable does not cache dependencies between builds. Each build will run on a fresh minion and as soon as the build finishes execution, minion will be deleted. However, we also understand that installing dependencies for each build will take more time and it affects your build speed . Hence we have a caching feature that helps you to cache dependencies between builds. Add the following line to your yml file to enable caching: 

.. code-block:: bash
  
     cache: true 

Before the build, we will check for the flag **cache: true** and if it exists, the minion will be cached after the build runs and the cached minion will be reused for further builds.  

You can use the **[reset_minion]** tag in commit message to reset the minion. We will clear all the cached dependencies and packages, when we see a [reset_minion] tag and your build will run on a fresh minion. Once this build finishes execution, we will cache the minion once again so that further builds can run using the cached minion.


---------

**Dedicated hosts**
------------------------

Shippable allows you to run builds using your own host machine. Build orchestration still happens through the multi-tenant service, but your builds are routed to your own hosts. This gives you complete control on your build system and also keeps your code in your machines. Dedicated hosts can be in the cloud or on premise.

The minimum requirements for a host are -

* 64 bit OS, Ubuntu 12.04 or 14.04

* 30GB HDD

* 2GB RAM

There is no CPU constraint, but builds will run slower on a slower CPU.

To use this feature, follow the steps mentioned `here <http://blog.shippable.com/dedicated-hosts->`_ and then update the **build_image** tag in your shippable.yml file with the path of your docker image. 

.. code-block:: python

   build_image: <docker_hub_username>/<image_name>

You can also specify docker run options like **privileged** and **network** in your yml file. The following network modes are supported:
 
- bridge - ( default) connect the container to the bridge 

- host - use the host's network stack inside the container 

Privileged is set to false by default. If you want to access all devices on the host, then you need to enable it using  **privileged: true** tag in your yml file. Configure your yml file as shown below to use host's network mode. 

.. code-block:: python

   build_image:
     name: <docker_hub_username>/<image_name>
     net: host
     privileged: true
  
 
