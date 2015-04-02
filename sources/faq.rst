:title: FAQ
:description: Commonly asked questions that will help with troubleshooting
:keywords: concepts, documentation, shippable, CI/CD

.. _faq:

FAQs
====
Having trouble with your builds? Here is a list of frequently asked questions.... hope this helps!

**Why can't I see some of my repositories in my Shippable account?**

This happens due to one of the following reasons:
1) You haven't enabled private repositories in your Shippable account. If this is the reason, please click on the 'Private repos off' icon on your dashboard and give us permissions to access your private repositories.
2) Your account hasn't yet been synced with the latest permissions from GitHub. To fix this, please click on the 'Sync account' icon on your dashboard.
3) You're a BitBucket user and you have mercurial repositories. We do not support mercurial at this time, so you will need to convert them to git or use another platform for CI.

**Why do I get an error when I try to enable a project that is listed on my dashboard?**

This usually happens if you are a collaborator on a project and the owner of the project has not given shippable access to the project. You can verify this by confirming that the owner of the project can see the project on their shippable dashboard.

**I have enabled my repository and committed code, but my build doesn't start. What could be wrong?**

Please check the 'Notifications' tab on your repository page on Shippable. If it shows any errors, fix those and try again.

If the error shows a parsing failure for the yml, you can validate the file at `YAML Lint <http://www.yamllint.com/>`_.

**Why can't I see my BitBucket repos in my Shippable account?**

Shippable only supports git based repositories, so if you have mercurial repositories in your BitBucket account, you will not see them in the Shippable repository list. If you cannot see git based repos, please open an issue on our [GitHub Support repo](https://github.com/Shippable/support).

**Why can't shippable see my org on github?**

Github's default policy when a new org is created is 'access restricted'. In order for Shippable to be able to see the org, you
must manually grant access to Shippable. This can be resolved by going to the third-party access section for the org, and clicking
'Remove restrictions' Under the 'Third-party application access policy' section.


**How do I link my github and bitbucket accounts?**

Shippable allows you to link both github and bitbucket service providers into a single account. Click on the bitbucket icon or github icon on the top right to link the respective account from the dashboard page.

For example: Sign in to shippable with your github account and click on the bitbucket icon on the top right to link your bitbucket account.

If you have already logged in to shippable with both the service providers account separately, then it will not allow you to link the accounts. You have to delete one of your shippable account and then click on the respective service provider icon from the other account. Deleting the account will also remove all its associated projects and builds, so first you need to decide which account you want to delete and then delete the account from the profile dropdown.

**Why am I not able to see BitBucket org repos after deleting and recreating my account on Shippable?**
 
Deleting the shippable account will also delete all the permissions associated with the account. If you recreate your account, bitbucket will not allow us to pull all the permissions you have, unless the owner of that organization logs in back to shippable and then click on the sync repos button to see the repos. 

**How do I set desired timezones inside the minions?**

By default, our minions are configured with ETC/UTC timezone which is set in /etc/timezone file for ubuntu minions. However, we allow you to set a specific time zone for the minion in before_script section of your yml file . For example, 

.. code-block:: python
        
   before_script:
     - echo 'Europe/Paris' | sudo tee /etc/timezone
     - sudo dpkg-reconfigure --frontend noninteractive tzdata

This will change your minion timezone to paris time. Refer the article `list of tz database time zones <http://en.wikipedia.org/wiki/List_of_tz_database_time_zones>`_  to select the timezone for your location.
 
