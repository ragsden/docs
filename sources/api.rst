:title: Shippable API
:description: How to interface with Shippable's API
:keywords: shippable, API, HTTP


.. _api: 

.. note::

  You must be on the Startup or Custom plan to use our API 

API
===========================

**Overview and Purpose**
---------------------------

The Shippable API enables you to do anything that you would normally 
do in the Shippable UI through HTTP requests. Our +API is RESTful, and 
can be interfaced through curl, third party tools, your own wrapper libraries, 
or any form of HTTPS communication.

Endpoint
^^^^^^^^^^^^^^^^^^^^^^^^^^
The main endpoint for interacting with our API is https://api.shippable.com

Authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^
Using our API requires authenticating with one of your Shippable API
tokens. Shippable API tokens can be generated from your account settings
page.

These tokens must be placed in the header of your HTTP request.
For example, if your API token has the value 10010, you would authenticate
this way with curl:

.. code-block:: bash

  curl -H "Authorization: apiToken 10010" https://api.shippable.com

A useful pattern is to set an env var with the value of your token.
For example, if we saved our token to the environment variable apiToken:

.. code-block:: bash

  curl -H "Authorization: apiToken $apiToken" https://api.shippable.com

This is useful not only because one no longer has to type type apiToken in
repeated times, but use of an env var allows for secure automatization of API 
scripts; it is dangerous to directly save your apiToken into code.

.. note::
 NEVER commit code containing your API token to a public repository. Doing
 so will compromise the security of your Shippable account. Treat your
 API token like a password

**Routes**
-----------------------------

/projects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The /projects endpoint will provide you with information about your projects, 
such as the projects on your Github or Bit Bucket account, and allow you to
retrieve information about them.

GET /projects
----------------------------------------------------------------


Will a return a list projects, and some info about the projects

Response

.. code-block:: javascript

  [
    {
      "mostRecentBuild": {
        "createdDate": "2015-02-22T02:52:00.526Z",
        "buildGroupNumber": 1,
        "id": "54e9444fac096311007dccd7",
        "status": 80,
        "durationCumulative": "29930",
        "commitSha": "39d50403945fb5a1d591b5ee2a549806ad80819a",
        "branch": "master"
      },
      "id": "54e80f5d91426fd6a78f6280",
      "language": "ruby",
      "autoBuild": true,
      "fullName": "user/example1",
      "name": "example1",
      "repositoryProvider": "github",
      "branches": [
        "master"
      ],
      "sourceDefaultBranch": "master"
    },
    {
      "mostRecentBuild": null,
      "branches": [],
      "repositoryProvider": "github",
      "name": "example2",
      "fullName": "user/example2",
      "autoBuild": false,
      "id": "54e3c35391426fd6a78cfd41"
    }
  ]
  
===================  =======      ==========================================================================
Name                 Type         Description
===================  =======      ==========================================================================
mostRecentBuild      object       An object of information about your most recent build, if available
id                   string       This project's unique id
language             string       The langauge of the project, as specified by the repo provider
autoBuild            boolean      States if the project will be auto built on pushes to the containing repo
fullName             string       The full name of the project, such as org/projectname
name                 string       A more succinct version of the fullName
repositoryProvider   string       The source providing the repo, such as Github or BitBucket
branches             list         A list of branches available to build from the repo
===================  =======      ==========================================================================

GET /projects/:projectId
----------------------------------------------------------------

Will return more in-depth information about the specified project.

Response

.. code-block:: javascript

  {
    "id": "54af3b7ld46123jfacaef00c",
    "branches": [
      "master",
      "feature1",
      "test"
    ],
    "autoBuild": true,
    "deployKey": {
      "public": "ssh-rsa SECRETE Shippable\n"
    },
    "settings": {
      "imageOptions": {
        "mounts": [],
        "ports": []
      },
      "environmentVariables": []
    },
    "created": "2015-01-09T02:23:49.586Z",
    "isEnabled": true,
    "enabledDate": "2015-02-09T06:40:25.463Z",
    "name": "project",
    "sourcePushed": "2015-03-17T15:22:00.000Z",
    "sourceCreated": "2015-01-06T05:05:22.000Z",
    "sourceUpdated": "2015-03-11T15:33:38.000Z",
    "language": "ruby",
    "updatedDate": "2015-03-18T23:29:19.334Z",
    "subscriptionId": "54af3b77d46935d5fbc1e00d",
    "sourceId": "28847632",
    "repositoryProvider": "github",
    "sourceRepoOwner": {
      "login": "owner",
      "starred_url": "https://api.github.com/users/owner/starred{/owner}{/repo}",
      "subscriptions_url": "https://api.github.com/users/owner/subscriptions",
      "organizations_url": "https://api.github.com/users/owner/orgs",
      "repos_url": "https://api.github.com/users/owner/repos",
      "events_url": "https://api.github.com/users/owner/events{/privacy}",
      "received_events_url": null,
      "type": "User",
      "site_admin": false,
      "gists_url": "https://api.github.com/users/owner/gists{/gist_id}",
      "following_url": "https://api.github.com/users/owner/following{/other_user}",
      "followers_url": "https://api.github.com/users/owner/followers",
      "html_url": "https://github.com/owner",
      "url": "https://api.github.com/users/owner",
      "gravatar_id": "",
      "avatar_url": "https://avatars.githubusercontent.com/u/184391?v=3",
      "id": 184391
    },
    "isFork": false,
    "isPrivateRepository": true,
    "sourceDefaultBranch": "master",
    "repositorySshUrl": "git@github.com:owner/project.git",
    "repositoryUrl": "https://api.github.com/repos/owner/project",
    "sourceDescription": "",
    "fullName": "owner/project"
  }


=====================================  ========      ==========================================================================
Name                                   Type           Description
=====================================  ========      ==========================================================================
id                                     string         This project's unique id
branches                               list           A list of branches available to build from the repo
autoBuild                              boolean        States if the project will be auto built on pushes to the containing repo
deployKey                              string         The ssh key used by shippable for deployments
settings                               object         Settings info for project, such as images and environment variables 
created                                string         When the project was created
isEnabled
enabledDate                                           The date the project was enabled on shippable for auto builds
name                                   string         A more succinct version of the fullName
sourcePushed
sourceCreated
sourceUpdated
language                               string         The langauge of the project, as specified by the repo provider
updatedDate                            string
subscriptionId                         string         The subscription id connected to this account
sourceId
repositoryProvider                     string         The source providing the repo, such as Github or BitBucket
sourceRepoOwner                        string         The owner/org that holds this repo
isFork                                 boolean
isPrivateRepository                    boolean        Specifies if the project is private or public
sourceDefaultBranch                    string         Specifies the default branch for the projec
repositorySshUrl                       string         The ssh url for the repo
repositoryUrl                          string         The web url for the project
sourceDescription
fullName                               string         The full name of the project, such as org/projectname
=====================================  ========      ==========================================================================


GET /projects/:projectId/runningBuilds
----------------------------------------------------------------

Returns a list of objects, where each object is a projection
of a build that is currently. 

Response

A list of :ref:`buildId` objects

GET /projects/:projectId/runningBuilds/:number
----------------------------------------------------------------

Returns a list of up to the specified number of 

Response

A list of :ref:`buildId` objects

GET /projects/:projectId/queuedBuilds
----------------------------------------------------------------

Returns a list of builds queued for this project.

Response

A list of :ref:`buildId` objects


GET /projects/:projectId/queuedBuilds/:number
----------------------------------------------------------------

Returns a list of up to the specified number of queued builds.

Response

A list of :ref:`buildId` objects

GET /projects/:projectId/recentBuilds
----------------------------------------------------------------

Returns a list of recent builds for the project. 

Response

A list of :ref:`buildId` objects

GET /projects/:projectId/recentBuilds/:number
----------------------------------------------------------------

Returns a list of up to the specified number of recent builds

Response

A list of :ref:`buildId` objects

/workflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
While /projects/* is used for retrieving info, /workflow/* is for initiating 
multi step processes, such as triggering or enabling a build, typically 
using your projectId as an input parameter.

POST /workflow/enableRepoBuild
----------------------------------------------------------------

This route is used for enabling your projects. It expects a JSON encoded
ProjectId.

Example Usage

.. code-block:: bash

  curl -H "Authorization: apiToken $apiToken" \ 
       -H "Content-Type: application/json" \
       -d "{\"projectId\": \"011d01\"}"
       https://api.shippable.com/workflow/enableRepoBuild

Query Parameters

========= ======== ===================
Name      Type     Description
========= ======== ===================
projectId string   Project's unique ID
========= ======== ===================


POST /workflow/disableBuild
----------------------------------------------------------------

Disable a repo from autobuilding

Query Parameters

========= ======== ===================
Name      Type     Description
========= ======== ===================
projectId string   Project's unique ID
========= ======== ===================

POST /workflow/cancelBuild
----------------------------------------------------------------

Cancels a build currently in progress

Query Parameters

========= ======== ===================
Name      Type     Description
========= ======== ===================
BuildId   string   Build's unique ID
========= ======== ===================


POST /workflow/triggerBuild
----------------------------------------------------------------

This route is used for starting builds of an enabled project

Query Parameters

=========== ======== ===================
Name        Type     Description
=========== ======== ===================
projectId   string   Project's unique ID
branchName  string   The branch to build
=========== ======== ===================

``branchName`` is an optional parameter. If it isn't specified, the default branch configured for the project in Github or Bitbucket will be built.

Response

.. code-block:: javascript

  {"BuildId": "aefjek3434j"}


===================  =======      ==========================================================================
Name                 Type         Description
===================  =======      ==========================================================================
BuildId              string       A build's unique Id
===================  =======      ==========================================================================



POST /workflow/validateDockerHubCredentials
----------------------------------------------------------------

Verifies a DockerHub account for the authenticated user

Query Parameters

========= ======== ====================
Name      Type     Description
========= ======== ====================
username   string   DockerHub username
password   string   DockerHub password
email      string   Dockerhub email
========= ======== ====================

/builds
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The builds endpoint allows you to get information from your
builds

.. _buildId:

/builds/:buildId
----------------------------------------------------------------

Contains information about the builds inside the specified build group,
along with associated metadata

Reponse

.. code-block:: javascript

  {
    "id": "55107b832132342fwlfjljf4",
    "isAutoCommit": false,
    "isAutoPush": false,
    "isAutoBuild": true,
    "isReRun": false,
    "isPullRequest": false,
    "isCompleted": true,
    "emailNotifications": [],
    "pullRequestNumber": null,
    "committer": {
      "avatarUrl": "https://avatars.githubusercontent.com/u/2321123?v=3",
      "displayName": "The Github User",
      "login": "github",
      "email": "user@gmail.com"
    },
    "lastAuthor": {
      "avatarUrl": "https://avatars.githubusercontent.com/u/2313232?v=3",
      "displayName": "Some githubuser",
      "login": "someuser",
      "email": "someuser@gmail.com"
    },
    "triggeredBy": {
      "avatarUrl": "https://avatars.githubusercontent.com/u/2323226?v=3",
      "email": "user@gmail.com",
      "displayName": "some user",
      "login": "user"
    },
    "requiresDedicatedHost": false,
    "builds": [
      {
        "id": "5510w323223rfwfb00e9b701",
        "size": null,
        "isCompleted": true,
        "isBuildCompleted": false,
        "canCommit": false,
        "matrixValues": [
          {
            "id": "55107b84142323f2f0e9b703",
            "value": "1.9.3",
            "name": "runtime"
          },
          {
            "id": "5510723f32f5780b00e9b702",
            "value": "",
            "name": "env"
          }
        ],
        "queuedDate": "2015-03-23T20:45:56.421Z",
        "isSubscriptionHost": false,
        "deprovisionStatusDate": "1970-01-01T00:00:00.000Z",
        "imageCommitStatusDate": "1970-01-01T00:00:00.000Z",
        "isFailureAllowed": false,
        "totalTests": 0,
        "testsFailed": 0,
        "testsPassed": 0,
        "testsSkipped": 0,
        "sequenceCoveragePercent": 0,
        "steps": {
          "commit": {
            "startTime": null,
            "duration": null,
            "endTime": null,
            "report": []
          },
          "upload": {
            "startTime": null,
            "duration": null,
            "endTime": null,
            "report": []
          },
          "purge": {
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:43:36.945Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:43:37.445Z"
              }
            ],
            "duration": 500,
            "startTime": "2015-03-23T20:43:36.945Z",
            "endTime": "2015-03-23T20:43:37.445Z"
          },
          "push": {
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:43:35.130Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:43:36.654Z"
              }
            ],
            "duration": 1524,
            "startTime": "2015-03-23T20:43:35.130Z",
            "endTime": "2015-03-23T20:43:36.654Z"
          },
          "cache": {
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:43:34.693Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:43:34.835Z"
              }
            ],
            "duration": 142,
            "startTime": "2015-03-23T20:43:34.693Z",
            "endTime": "2015-03-23T20:43:34.835Z"
          },
          "report": {
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:46:42.256Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:46:43.694Z"
              }
            ],
            "duration": 1438,
            "startTime": "2015-03-23T20:46:42.256Z",
            "endTime": "2015-03-23T20:46:43.694Z"
          },
          "boot": {
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:42:56.359Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:43:11.246Z"
              }
            ],
            "duration": 14887,
            "startTime": "2015-03-23T20:42:56.359Z",
            "endTime": "2015-03-23T20:43:11.246Z"
          },
          "dequeue": {
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:42:47.069Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:42:47.315Z"
              }
            ],
            "duration": 246,
            "startTime": "2015-03-23T20:42:47.069Z",
            "endTime": "2015-03-23T20:42:47.315Z"
          },
          "pull": {
            "startTime": "2015-03-23T20:42:47.653Z",
            "duration": 8358,
            "endTime": "2015-03-23T20:42:56.011Z",
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:42:47.653Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:42:56.011Z"
              }
            ]
          },
          "build": {
            "startTime": "2015-03-23T20:43:11.562Z",
            "duration": 20764,
            "endTime": "2015-03-23T20:43:32.326Z",
            "report": [
              {
                "status": 20,
                "time": "2015-03-23T20:43:11.562Z"
              },
              {
                "status": 30,
                "time": "2015-03-23T20:43:32.326Z"
              }
            ]
          }
        },
        "environment": "",
        "gemfile": null,
        "jdk": null,
        "version": "1.9.3",
        "status": 30,
        "buildNumber": 1,
        "commitTag": "artifact.22.1",
        "startDate": "2015-03-23T20:45:56.474Z",
        "hostId": "54ee2d61b6e6dd0d0037a124",
        "consoleLogLineCount": 351,
        "consoleLogBytes": 13593,
        "isArchiveAvailable": true,
        "endDate": "2015-03-23T20:46:47.189Z",
        "duration": 50715,
        "branchCoveragePercent": 0
      }
    ],
    "createdDate": "2015-03-23T20:45:56.421Z",
    "updatedDate": "2015-03-23T20:45:56.421Z",
    "repositorySize": 152940,
    "repositoryFileCount": 1,
    "shouldArchive": true,
    "privileged": false,
    "imageName": "shippable/minv2",
    "imageId": "540ef25d5e5bad45f3fa6cb7",
    "settings": {
      "imageOptions": {
        "networkMode": "bridge",
        "privileged": false
      },
      "imageId": "540ef25d5e5bad45f3fa6cb7",
      "runCommand": "",
      "imageName": "shippable/minv2"
    },
    "language": "ruby",
    "compareUrl": "https://github.com/user/project/compare/0000000000000000000000000000000000000001...1e1616bad7ab7206081cb83d3c97c0e53c9cb365",
    "beforeCommitSha": "0000000000000000000000000000000000000000",
    "branch": "artifact",
    "branchHead": "artifact",
    "status": 30,
    "buildGroupNumber": 22,
    "projectId": "54e846935ab6cc13528d3d62",
    "lastCommitShortDescription": "artifact",
    "commitSha": "1e1616bad7ab7206081cb83d3c97c0e53c9cb365",
    "commitUrl": "https://github.com/user/project/commit/1e1612f23f32f206081cb83d3c97c0e53c9cb365",
    "createdByAccountId": "540e74943999311130416dd0",
    "baseCommitRef": "",
    "runCommand": "",
    "emailOnSuccess": "change",
    "emailOnFailure": "always",
    "timeoutMS": 5400000,
    "buildRunnerVersion": "1.0.0",
    "statusMessage": "SUCCESS",
    "durationCumulative": "50715",
    "shouldDecryptSecureEnvs": true,
    "branchCoveragePercent": 0,
    "sequenceCoveragePercent": 0,
    "testsSkipped": 0,
    "testsPassed": 0,
    "testsFailed": 0,
    "totalTests": 0,
    "parallelizedTest": false,
    "network": "bridge"
  }

======================   =========      =========================================================================================
Name                     Type           Description
======================   =========      =========================================================================================
id                       string         A build groups unique id
isAutoCommit             boolean        Set to true if caching is enabled
isAutoPush               boolean        From the project settings page or commit_container tag. Will do a docker push if true.
isAutoBuild              boolean        Set to true if this build was triggered by a webhook.
isReRun                  boolean        Set to true if this is a manually triggered rerun of the build
isPullRequest            boolean        Set to true if this build was triggered by a pull request
isCompleted              boolean        Set to true if this build has completed
emailNotifications       list           A list of emails to notify about the builds
pullRequestNumber        number         Number of the pull request
committer                object         An object of information about the commiter
lastAuthor               object         An object of information about the last author
triggeredBy              object         An object of information about the triggerer
requiresDedicatedHost    boolean        Set to true if this build has features that require a dedicated host
builds                   list           A list of builds that are created by this build group
createdDate              string         The date the repo was created
updatedDate              string         The date of the last time the repo was updated
repositorySize           number         The total size of all the files in the repo
repositoryFileCount      number         The number of files in the repo
shouldArchive            boolean        Set to true if bulid archiving is enabled
privileged               boolean        Set to true if the docker container is running in privileged mode
imageName                string         The name of the docker image used to run this build
imageId                  string         The id of the docker image used to run this build
settings                 object         An object containing the settings for this project
language                 string         The programming language for this project
compareUrl               string         A link to the url containing the comparison to the last commit
beforeCommitSha          string         The SHA of the project before the commit
branch                   string         The git branch that this build group is based on
branchHead               string         The head of the branch
status                   string         Status on the build group, such as if it's finished or failed
buildGroupNumber         string         The number of this build group
projectId                string         The id of the project
lastCommitShortDescri    string         A truncated description of the last commit
commitSha                string         The computed SHA of the commit that generated this build group derives from
commitUrl                string         The url where the commit can be found
createdByAccountId       string         The account that created the project
baseCommitRef            string         A string for the ref to the base commit
emailOnSuccess           boolean        Sets if email notifications will be sent on build success
emailOnFailure           boolean        Sets if email notifications will be sent on build failure
timeoutMS                number         How long in miliseconds the build runs before timing out.
buildRunnerVersion       number         The version of the build runner
statusMessage            string         The current status of the build
durationCumulative       number         The cumulative duration of all the builds
shouldDecryptSecureEn    boolean        True if there are encrypted env variables used in the shippableyml file
branchCoveragePercent    number         The percentage of branches (if/then/else condtions) that are covered by tests
sequenceCoveragePerce    number         Percentage of lines there are code coverage for
testsSkipped             number         The number of tests that were skipped
testsPassed              number         The number of tests that passed
testsFailed              number         The number of tests that failed
totalTests               number         The total number of tests.
parallelizedTest         boolean        Set to true if the test was set to be parallelized
network                  string         The network settings for the docker container
======================   =========      =========================================================================================

/builds/:buildId/:buildItemNumber/artifacts
----------------------------------------------------------------

Returns a url to a tarball containing your build artifacts

Response

https://prod-shippable.s3.amazonaws.com/artifacts/subscriptions/.../tar.gz


/accounts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

GET /accounts
----------------------------------------------------------------

Returns a string list of your account ids

Response

.. code-block:: javascript
  
  ["322fasf323f3gw3"]


GET /accounts/:accountId
----------------------------------------------------------------

.. code-block:: javascript

  {
    "id": "640e74943999391400416qr0",
    "lastUsedIdentityId": "640e74943999391400416qr1",
    "identities": [
      {
        "id": "640e74943999391400416qr1",
        "scopes": [
          "read:org",
          "repo",
          "user:email",
          "write:repo_hook"
        ],
        "enforceScopes": [
          "read:org",
          "repo",
          "user:email",
          "write:repo_hook"
        ],
        "emails": [
          {
            "primary": true,
            "verified": true,
            "email": "user@gmail.com"
          }
        ],
        "migratedProviderId": true,
        "providerOwnedPrivateRepos": null,
        "providerType": "User",
        "providerId": "2054256",
        "provider": "github",
        "avatarId": "",
        "avatarUrl": "https://avatars.githubusercontent.com/u/RRR4256?v=3",
        "displayName": "A user",
        "userName": "user",
        "email": "",
        "providerBlog": "",
        "providerCompany": "",
        "providerLocation": "",
        "providerFollowerCount": 3,
        "providerPrivateGists": null,
        "providerPublicRepoCount": 48,
        "providerPublicGistCount": 2,
        "providerTotalPrivateRepos": null
      }
    ],
    "systemRoles": [
      "user"
    ],
    "created": "2014-09-09T03:31:32.951Z"
  }

===================================== ======== ===================================
Name                                  Type     Description
===================================== ======== ===================================
id                                    string   Account ID
lastUsedIdentityId                    string   id of last used identity. 
identities                            list     A list of this accounts identitiesj
created                               string   When the account was created
===================================== ======== ===================================

DELETE /accounts/:accountId
----------------------------------------------------------------

Deletes the specified account

GET /accounts/:accountId/identities
----------------------------------------------------------------

A string list of identity ids associated with this account. Your account can have multiple
identities. There will always be at least one identity, and that is the identity
of your linked github or bitbucket account. Another identity your account could take
is the identity of an organization it belongs to.

Response

.. code-block:: javascript
  
  ["322fasf323f3gd1", "322fasf323f3gd2"]

