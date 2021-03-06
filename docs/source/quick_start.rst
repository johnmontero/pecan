.. _quick_start:

Quick Start
===========

Let's create a small sample project with Pecan.

.. note::
    This guide does not cover the installation of Pecan. If you need
    instructions for installing Pecan, go to :ref:`installation`.


Base Application Template
-------------------------

A basic template for getting started is included with Pecan.  From
your shell, type::

    $ pecan create

The above command will prompt you for a project name. I chose *test_project*,
but you can also provided as an argument at the end of the example command
above, like::

    $ pecan create test_project

Go ahead and change into your newly created project directory::

    $ cd test_project
    $ ls

This is how the layout of your new project should look::

    ├── MANIFEST.in
    ├── config.py
    ├── public
    │   ├── css
    │   │   └── style.css
    │   └── javascript
    │       └── shared.js
    ├── setup.cfg
    ├── setup.py
    └── test_project
        ├── __init__.py
        ├── app.py
        ├── controllers
        │   ├── __init__.py
        │   └── root.py
        ├── model
        │   └── __init__.py
        ├── templates
        │   ├── error.html
        │   ├── index.html
        │   ├── layout.html
        │   └── success.html
        └── tests
            ├── __init__.py
            ├── test_config.py
            └── test_root.py

The amount of files and directories may vary, but the above structure should
give you an idea of what you should expect.

A few things have been created for you, so let's review them one by one:

* **public**: All your static files (like CSS and Javascript) live here. If you
  have any images (this example app doesn't) they would live here too.


The remaining directories encompass your models, controllers and templates, and
tests:

*  **test_project/controllers**:  The container directory for your controller files.
*  **test_project/templates**:    All your templates go in here.
*  **test_project/model**:        Container for your model files.
*  **test_project/tests**:        All of the tests for your application.

To avoid unneeded dependencies and to remain as flexible as possible, Pecan
doesn't impose any database or ORM (Object Relational Mapper) out of the box. 
You may notice that **model/__init__.py** is mostly empty.  You may wish to add 
code here to define tables, ORM definitions, and parse bindings from your 
configuration file.


.. note::
    The base project contains some ready-to-run tests. Try running
    ``py.test`` (the recommended test runner for Pecan) and watch them pass!


.. _running_application:

Running the application
-----------------------
Before starting up your Pecan app, you'll need a configuration file.  The
base project template should have created one for you already, ``config.py``.

This file already contains the necessary information to run a Pecan app, like
ports, static paths and so forth. 

If you just run ``pecan serve``, passing ``config.py`` as an argument for
configuration, it will bring up the development server and serve the app::

    $ pecan serve config.py 
    Starting subprocess with file monitor
    Starting server in PID 000.
    serving on 0.0.0.0:8080 view at http://127.0.0.1:8080

    
The location for the config file and the argument itself are very flexible - 
you can pass an absolute or relative path to the file.


Simple Configuration
--------------------
For ease of use, Pecan configuration files are pure Python.

This is how your default configuration file should look::

    from test_project.controllers.root import RootController

    import test_project

    # Server Specific Configurations
    server = {
        'port' : '8080',
        'host' : '0.0.0.0'
    }

    # Pecan Application Configurations
    app = {
        'root' : RootController(),
        'modules' : [test_project],
        'static_root' : '%(confdir)s/public', 
        'template_path' : '%(confdir)s/test_project/templates',
        'reload': True,
        'debug' : True,
        'errors' : {
            '404' : '/error/404',
            '__force_dict__' : True
        }
    }

    # Custom Configurations must be in Python dictionary format::
    #
    # foo = {'bar':'baz'}
    # 
    # All configurations are accessible at::
    # pecan.conf


**Nothing** in the configuration file above is actually required for Pecan to
run. If you fail to provide some values, Pecan will fill in the missing things
it needs to run.

You can also add your own configuration as dictionaries.

For more specific documentation on configuration, see the :ref:`Configuration`
section.

    
Root Controller
---------------
The Root Controller is the root of your application.

This is how it looks in the project template::

    from pecan import expose, request
    from formencode import Schema, validators as v
    from webob.exc import status_map


    class SampleForm(Schema):
        name = v.String(not_empty=True)
        age = v.Int(not_empty=True)


    class RootController(object):
        @expose('index.html')
        def index(self, name='', age=''):
            return dict(errors=request.validation_errors, name=name, age=age)
        
        @expose('success.html', schema=SampleForm(), error_handler='index')
        def handle_form(self, name, age):
            return dict(name=name, age=age)
        
        @expose('error.html')
        def error(self, status):
            try:
                status = int(status)
            except ValueError:
                status = 0
            message = getattr(status_map.get(status), 'explanation', '')
            return dict(status=status, message=message)



You can specify additional classes if you need to do so, but for now we have an
*index* and *handle_form* method.

**index**: is *exposed* via the decorator ``@expose`` (which in turn uses the
``index.html`` template) at the root of the application (http://127.0.0.1:8080/),
so anything that hits the root of your application will touch this method.

Notice that the index method returns a dictionary - this dictionary is used as
a namespace to render the specified template (``index.html``) into HTML.

Since we are performing form validation and want to pass any errors we might
get to the template, we set ``errors`` to receive form validation errors that
may exist in ``request.validation_errors``.


**handle_form**: receives 2 arguments (*name* and *age*) that are validated
through the *SampleForm* schema.

The ``error_handler`` has been set to index.  This means that when errors are
raised, they will be sent to the index controller and rendered through its
template.

**error**: Finally, we have the error controller that allows your application to 
display custom pages for certain HTTP errors (404, etc...).

Application Interaction
-----------------------
If you still have your application running and you visit it in your browser,
you should see a page with some information about Pecan and a form so you can
play a bit.
