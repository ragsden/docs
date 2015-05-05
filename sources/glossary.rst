:title: Glossary
:description: Definitions of terms used in Shippable documentation
:keywords: concepts, documentation, shippable, CI/CD

.. _glossary:

GLOSSARY
========

Definitions of terms used in Shippable documentation


**Subscriptions**
-----------------
A default subscription is created for every user account so that Shippable can group CI projects under a common entity. A default free subscription is created on signup with quotas as per our pricing plans. Users can upgrade this free subscription to a paid plan and receive higher quotas.


**Minions**
-----------
Minions are containers that will run your builds and tests. The more minions you have, the more builds you can run in parallel.  Each minion starts from a base image and can be further customized by specifying before_install scripts in the YML file.


**Projects**
------------
A Project is used to handle builds/CI/CD on your repo. You need to create a project for each repo you want CI enabled. Every project is under a subscription and your limits/quotas are tracked at the subscription level.
