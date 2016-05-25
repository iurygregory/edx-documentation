.. _edx_user_interfaces:

##############################
Developing User Interfaces
##############################

This section provides information about developing user interfaces (UI) for edX
applications.

All new edX pages should be composed of elements and styling shown in the edX
Pattern Library. You can use JavaScript from the edX UI Toolkit JavaScript
library to implement pages in a way that is consistent with edX UI
architecture.

.. contents::
 :local:
 :depth: 2


.. 20160518 - any important information about UI development should be moved
.. from the wiki to the developer's guide here.

The Open edX wiki includes additional details about developing edX user
interfaces. For more information, see `edX Front End Development
<https://openedx.atlassian.net/wiki/display/FEDX/edX+Front+End+Development>`_.

.. _pattern_library:

*****************************
Using the edX Pattern Library
*****************************

The edX Pattern Library is a collection of web application
components and visual styles that you can use to develop UIs that
are consistent with the edX platform.

The edX Pattern Library is available at `ux.edx.org`_.

The edx-platform GitHub repository includes an `example web page
<https://github.com/edx/edx-platform/blob/master/lms/templates/ux/reference
/pattern-library-test.html>`_ that demonstrates using the Pattern Library. You
can use the example page as a template for creating your own new pages.

.. _ui_toolkit:

************************
Using the edX UI Toolkit
************************

The edX UI Toolkit is a JavaScript library that you can use to
develop UI functionality for the edX platform. The edX UI Toolkit
is available from the `edx-ui-toolkit GitHub repository`_ . For more
information about the UI Toolkit, see `ui-toolkit.edx.org`_ .

For more information about writing JavaScript for edX UIs, see :ref:`edx_javascript_guidelines`.

.. _adding_a_new_ui_page:

********************
Adding a New UI Page
********************

This document is a collection of things to think about when creating a new page
in Studio or the LMS. This is a living document so please add to it if you find
something to be missing.

You should also read `Code Review Guidelines and Gotchas <https://openedx.atlas
sian.net/wiki/display/TNL/Code+Review+Guidelines+and+Gotchas>`_ to understand
more general best practices, as well as how to submit your new code for review.

.. contents::
 :local:
 :depth: 2

==========================
Use the Pattern Library
==========================

All new pages should use the edX Pattern Library. There are a few
considerations:

* Use the same main.html template as for non-Pattern Library pages, but then
  pass the two following context parameters.

    .. code-block:: none

        {
            ...
            'disable_courseware_js': True,
            'uses_pattern_library': True,
        }

* Create two new root sass files for your page (for ltr and rtl).

    * Note that the RTL version of the file must have the same filename as the
      LTR version but ending with ``-rtl``.

    * Declare the name of your sass files at the top of your main Mako file.
      For example, if your file is feature-main.scss:

        .. code-block:: html

            <%! main_css = "feature-main" %>

Examples:

* `Pattern Library test page <https://github.com/edx/edx-
  platform/blob/master/lms/templates/ux/reference/pattern-library-test.html>`_

* PR to convert the programs list to be a Pattern Library page: `PR 12380
  <https://github.com/edx/edx-platform/pull/12380>`_

======================================
Use JavaScript Instead of CoffeeScript
======================================

The edX front end code base is currently a mixture of CoffeeScript and
JavaScript, and we have decided to standardize on JavaScript for new work.
There are a number of reasons for this decision:

* It is hard to jump back and forth between the two syntaxes.

* It adds a compilation step which slows down iterative development.

* Most third party libraries are implemented and hence documented in
  JavaScript.

* Client-side debugging is done in JavaScript which makes it hard to correlate
  back to the original code.

* The diff-cover tools do not understand CoffeeScript so code coverage is not
  tracked.

* A lot of the traditional benefits of CoffeeScript can now be achieved using
  JavaScript libraries.

All new front end development should be done in JavaScript, while legacy
CoffeeScript code will be converted over on an as-needed basis. For pragmatic
reasons, targeted changes to CoffeeScript will still be accepted.

For more information about writing JavaScript for edX UIs, see
:ref:`edx_javascript_guidelines`.

.. _use_requirejs_to_manage_file_dependencies:

=========================================
Use RequireJS to Manage File Dependencies
=========================================

All new pages should use `RequireJS <http://requirejs.org/>`_ to manage loading
of file dependencies.

* Use the standard syntax to load JavaScript dependencies.

  .. note::

      For the LMS, you have to wrap each call to define or require in
      order for it to work with the namespaced version of RequireJS.

  .. code-block:: javascript

    ;(function (define) {
        'use strict';
        define([...], function (...) {
            ...
        });
    }).call(this, define || RequireJS.define);

* Use RequireJS Text for your templates (see
  :ref:`requirejs_optimization_javascript` for details).

* Create a factory class that is included by your Mako template using
  require_module.

    * This Mako function ensures that the factory is loaded correctly for both
      the optimized and non-optimized pipeline.

    * When accessed in the optimized mode, the query parameter ``?raw`` will be
      added to the URL to ensure that the file doesn't get post-processed.

    * Note: ``require_module`` is not needed on Studio currently but will be
      when the two optimizer pipelines are unified.

* If you need server-side data passed to your page, pass it as one or more
  parameters to your factory.

    * To pass JSON data structures, use json.dumps but be sure to escape it
      with ``EscapedEdxJSONEncoder`` to avoid XSS.

* Be careful when adding new third party libraries.

    * If the library doesn't support RequireJS then it needs to be shimmed in
      require-config.js.

* If the dependency is pre-loaded, then be sure to specify its path as empty in
  build.js so that the optimizer doesn't include it in the bundled file. This
  is especially important for files that are used by both by JavaScript
  leveraging RequireJS and "non-RequireJS JavaScript" within the same page.

  .. code-block:: javascript

    paths: {
        'gettext': 'empty:',
        'jquery': 'empty:',
        ...
    }

Here is an example Mako code block that demonstrates how this all ties
together.

.. code-block:: html

    <%block name="js_extra">
    <%static:require_module module_name="teams/js/teams_tab_factory"
      class_name="TeamsTabFactory">
        TeamsTabFactory(
            $('.teams-content'),
            ${ json.dumps(topics, cls=EscapedEdxJSONEncoder) },
            '${ topics_url }',
            '${ unicode(course.id) }'
        );
    </%static:require_module>
    </%block>

Here are a few examples.

* `Studio Course Outline (Studio) <https://github.com/edx/edx-
  platform/blob/master/cms/static/js/views/pages/course_outline.js>`_

* `Courseware Search (LMS) <https://github.com/edx/edx-platform/blob/master/lms
  /static/js/search/course/views/search_results_view.js>`_

* `Student Notes (LMS) <https://github.com/edx/edx-
  platform/blob/master/lms/static/js/edxnotes/views/notes_factory.js>`_

* `Teams courseware tab (LMS) <https://github.com/edx/edx-platform/blob/master/
  lms/djangoapps/teams/static/teams/js/views/teams_tab.js>`_

.. note::

    Using RequireJS in XBlocks is not currently supported. This is something
    that we intend to address in the future.

.. _use_underscore_and_requirejs_text:

===========================================================
Use Underscore and RequireJS Text for Client-Side Templates
===========================================================

We have standardized on using Underscore for all client-side templates. There
are some things to consider.

* Add your templates to a static/templates directory in your Django app.

* If you need mock HTML for Jasmine, put them in a mock subdirectory.

* Use :ref:`RequireJS Text <use_underscore_and_requirejs_text>` to load your
  templates (note that most code today statically includes the template in the
  page).

RequireJS Text has several benefits over the old mechanism of statically
including the templates in the Mako-generated HTML.

* Mako templates no longer need to explicitly include every needed template in
  the HTML.

* The template lookup is much faster when optimized as it doesn't have to
  extract it from the DOM. For developers, the template is fetched
  asynchronously like any other RequireJS dependency.

* The templates get included in the minified .js file for the page so the
  content is only downloaded once and then cached in the browser. Including the
  template in the Mako template causes it to be downloaded every time.

* Unit tests don't need to load all the templates into fixtures for the views
  to work, as the view will load its template dependencies directly.

Here are a few examples.

* `Paging Header component <https://github.com/edx/edx- platform/blob/master/co
  mmon/static/common/js/components/views/paging_header.js>`_

* `Teams topic card <https://github.com/edx/edx-
  platform/blob/master/lms/djangoapps/teams/static/teams/js/views/topics.js>`_

.. _requirejs_optimization_javascript:

============================================================
Ensure That RequireJS Optimizer Can Optimize Your JavaScript
============================================================

We use `RequireJS Optimizer <http://requirejs.org/docs/optimization.html>`_ to
optimize the JavaScript that we use in production. This works as part of the
production pipeline to merge all of a page's JavaScript and templates into a
single minified file. This greatly reduces the number of files that the browser
needs to request, as well as the number of bytes that need to be fetched.
Furthermore, in LMS un-optimized RequireJS files are never cached, leading to
significant performance degradation.

The basic approach is to move all of the page view construction logic into a
single factory file. This factory is responsible for creating the models and
views required to render the page. The idea is that the optimizer can produce a
single minified file for the factory by statically determining all of its
dependencies.

* Structure your page so that it has a single root factory file (see
  :ref:`use_requirejs_to_manage_file_dependencies`).

  Be sure to use ``require_module`` to load the factory as described above.

* List your factory in the RequireJS Optimizer build file.

    * `Studio build file <https://github.com/edx/edx-
      platform/blob/master/cms/static/build.js#L24>`_

    * `LMS build file <https://github.com/edx/edx-
      platform/blob/master/lms/static/lms/js/build.js>`_

* By default, devstack runs with unoptimized files so that changes are picked
  up immediately.

  To run with optimized files, specify the ``--optimized`` parameter to Paver's
  ``devstack`` command:

  .. code-block:: bash

    paver devstack lms --optimized

* Note that Bok Choy tests run with optimized files to verify that they are
  being generated as expected.

* Be sure to verify your page on a sandbox, checking that all dependencies are
  included in the optimized factory file.

    * For LMS files, take care that all of the JS files have a MD5 hash code
      between their filename and the js extension-- files without MD5 hash
      codes will not be cached in production and are indicative of incorrect
      optimization.

    * Studio RequireJS files currently do not use the MD5 hashing mechanism,
      and instead store files within a unique directory that changes with each
      release. We hope to change this in the future.

=================================
Use Backbone to Build Your New UI
=================================

Code written with Backbone is more modular, more extensible, and more easily
unit tested with Jasmine.

See `How to add a new feature to LMS or Studio <https://openedx.atlassian.net/w
iki/display/AC/How+to+add+a+new+feature+to+LMS+or+Studio>`_ to see how to
structure your Backbone code.

===================================
Set Up Your Page To Use Online Help
===================================

.. note::

    Online help is currently for Studio only. See  :jira:`TNL-622`  for the
    story to add help to the LMS.

#. Ask a documentation team member for a help token for your new page.

#. At the top of your Mako template, add the following element.

    .. code-block:: html

        <%def name="online_help_token()"><% return "YOUR_NEW_TOKEN" %></%def>

#. Update edx-platform/docs/config.ini to reference the new token (or have the
   doc team do so when they have the target html file ready)

#. If you want a direct link other than the standard "Help" button, use the
   following URL:

    .. code-block:: html

    <a href="${get_online_help_info(online_help_token())['doc_url']}"
       target="_blank" class="button external-help-button">${_("Learn more
       about MY_FEATURE")}</a>

=======================================
Verify the Performance of Your New Page
=======================================

All new pages should have the performance analyzed to ensure that they perform
adequately.

See `Measuring edx platform performance with sitespeed.io <https://openedx.atla
ssian.net/wiki/display/PERF/Measuring+edx+platform+performance+with+sitespeed.i
o>`_ for a description of the recommended approach.



.. include:: ../../links/links.rst
