:title: nodejs
:description:  language specific guide for node_js
:keywords: nodejs, mocha, npm, node

.. _langnodejs:

Nodejs 
======

This section helps you to configure the yml file for your node_js project.

**Node_js versions for testing** :


- Set the appropriate language and the version number. You can test against multiple versions for a single push by adding more entries. node_js minions use ``node_js`` by default to set the version.
    .. code-block:: bash
	
	# language setting
	language: node_js
	
	# version numbers
	node_js:
  	  - "0.11"
	  - "0.10"
          - "0.8"
          - "0.6"

.. note::
 We are setting multiple versions of node_js here which means a single push to repo will trigger multiple builds. 


**Test scripts**

- To run your test suites using NPM, specify it using script key.  
	.. code-block:: bash
		
		script: npm test

-  You can also add testing frameworks like Vows, Expresso in the package.json file.

-  To test using Vows:
	.. code-block:: python 	

            "scripts": {
              "test": "vows --spec"
        	} 

-  To test using Expresso:
	.. code-block:: python
	    
             "scripts": {
               "test": "expresso test "
        	}

-  You can also install your project dependencies using `NPM <http://npmjs.org/>`_
	.. code-block:: python
	   
              node_js:
	          - "0.10"
	      before_install: npm install -g grunt-cli
      

- If you want to build a project with node versions like 0.6, 0.8, 0.10, and 0.11 and want to use the same package.json, add the following line to your yml file, which will upgrade the npm to v.1.4 for node versions 0.6 and 0.8.
      .. code-block:: python

           if [[ $SHIPPABLE_NODE_VERSION =~ [0].[6-8] ]]; then npm install -g npm@~1.4.6; fi


-  Keep the output of test and code coverage generated in the Shippable folders to get the visualization of your reports.

-  If your test runner uses the junit format, then you can save the output in shippable/testresults folder so that shippable can parse the test reports. You can also save the output of code coverage in shippable/codecoverage folder so that shippable can parse the coverage reports.

**Build Examples**

Want to see build examples? Reference our :ref:`Nodejs build samples <Nodejs>`.
