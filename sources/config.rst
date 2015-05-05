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
  
 
