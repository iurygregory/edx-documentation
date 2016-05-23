..  _edx_python_guidelines:

##########################
EdX JavaScript Style Guide
##########################

This section describes the requirements and conventions used to contribute
JavaScript programming to the edX platform.

.. contents::
 :local:
 :depth: 2

****************
Google Standards
****************

In the `Google JavaScript Style Guide <http://google-
styleguide.googlecode.com/svn/trunk/javascriptguide.xml>`_ the section
`JavaScript Language Rules <http://google-styleguide.googlecode.com/svn/trunk/j
avascriptguide.xml#JavaScript_Language_Rules>`_ has very reasonable "tips" on
how one should and shouldn't write JavaScript code. Since we don't want to re-
invent the wheel and come up with and maintain our own unique set of standards,
we have adopted these as our go-to standards.

**********
Code Style
**********

Use 4 spaces for indenting

.. code-block:: javascript

    function renderElements(state) {
        var youTubeId;

        if (state.videoType === 'html5') {
            state.videoPlayer.player = new HTML5Video.Player(state.el, {
                playerVars: state.videoPlayer.playerVars,
                videoSources: state.html5Sources,
                events: {
                    onReady: state.videoPlayer.onReady,
                    onStateChange: state.videoPlayer.onStateChange
                }
            });
        }
    }

****************
Linting
****************

Use JSHint with the following settings: TDB

****************
Good Practices
****************

All local variables in a function should be declared first, before any code statements. Even if initialization happens later on, the variable should be declared. This is benefiter for quickly checking which variables are local and which are defined further up the scope chain (or are global). One doesn't have to skim through the entire function body searching for the var keyword to discover local variable declarations - they are all in one place at the top. This also follows the logic of hoisting in JavaScript. By placing all variable declarations at the top, you are always aware of the fact that all variables are declared at the time of function's invocation, and by default assigned the value of undefined. Even if the variable definition happens somewhere in the middle of the function's body, the variable declaration is silently "hoisted" to the top of the function by the compiler.

Think carefully when to use function declaration and when to use function expression. If the function will be invoked throughout the code, then it is probably a safe bet to use a function declaration. Due to hoisting, it will be immediately available (either globally, or in the scope of the function), even if declared at the end of the source file or function body. However, if the function will be used in some cases, then a function expression is more suitable. The function will be "created" only when AND IF the compiler execution flow reaches the function expression statement. Memory will be allocated only when necessary. When developing a new applet, internal function API is generally hidden from everyone else. This means that one is pretty sure with what parameter's and their types functions will be invoked with. This means that it is redundant to do type checking or other double-safety things on function's parameters. However, in the initial stage of development, it is good to check. For this it is best to write tests (see Jasmine, unit testing).

When creating an object and populating it with properties, the names of the properties should be surrounded by quotes only if they include some unsupported symbols. Otherwise, for code consistency, leave off the quotes (single or double).

Sometimes it is very tempting to test if an object really does contain a property by using the ``Object.hasOwnProperty()`` method. This is unnecessary, and should be done only in the case when it has been confirmed that the prototype chain of the object also has a property with the same name.

When writing a JavaScript component, please adhere to the following guidelines:

* Write modular JavaScript whenever possible
* Use closures whenever possible
* Lint your code

******************
Modular JavaScript
******************

A module is a function or object that presents an interface but that hides its state and implementation. Functions are used to produce modules, eliminating global variables and increasing security and privacy.

Modular JavaScript also allows us to bundle complete functionality which makes using RequireJS possible.

Example:

.. code-block:: javascript

    var SomeObject = {

        add: function(int1, int2) {
            return Number(int1) + Number(int2);
        },

        result: function() {
            this.add(2, 10);  // returns 12
        }
    };

******************
Closures
******************

In addition to writing modules, enclosing your JavaScript in a closure will greatly increase security. A closure returns an object literal, and uses scope to keep the methods hidden.

Example:

.. code-block:: javascript

    var someObject = (function() {

        var value = 0;

        return {
            add: function(int1, int2) {
                return Number(int1) + Number(int2);
            },

            result: function() {
                value = this.add(2, 10);  // returns 12
            }
        };
    }());

******************
Linting
******************

We want our JavaScript to be properly written and error-free. Luckily this isn't something you need to worry about manually including because we include a linter in our grunt compile. So whenever you fire up your local pattern library install, your JavaScript will be linted automatically. Check for any errors or warnings in your console!

********************
RequireJS
********************

We use RequireJS to help manage JavaScript dependencies and integrate our pattern library more efficiently into our main platform, since it too uses RequireJS. You should have some familiarity with using RequireJS so that your scripts are not only included, but efficient.

===========================
Brief overview of RequireJS
===========================

RequireJS makes use of required files and dependencies for every individual script. Any script you need to be loaded at site load should be included in the required.js file, which is loaded with Require (and other than Require, is the only file loaded at site load).

Example:

.. code-block:: javascript

    require([
        'jquery',
        '/public/pldoc/js/ui.js',
        '../../js/svg4everybody.min'
        ],
        function($, Ui) {

        ...your code here...
    });

This is our required.js file. It sets the required files at site load and then sets aliases, if desired. Our Ui file here, loads our other front-end scripts.

.. code-block:: javascript

    define([
        'jquery',
        '/public/pldoc/js/jquery.smooth-scroll.js',
        '/public/pldoc/js/tabs.js',
        '/public/pldoc/js/size-slider.js',
        '/public/pldoc/js/color-contrast.js',
        '/public/js/select-replace.js'
        ], function($, smoothScroll, Tabs, IconSlider, ColorContrast) {

        ...your code here...
    });

At the top of our ui.js file, here's where we load the rest of our front-end files. Similar to our required.js file, we reference the files and then give them an alias which we may use elsewhere.

===========================
Adding a new JavaScript
===========================

When you want to add a new JavaScript, make it modular and use a closure. When that's done, add the RequireJS wrapper, like so:

.. code-block:: javascript

    define([
        'jquery'
        ], function($) {

        ...your code here...

    });

Your define block should include any dependencies your file has. In this case, this particular script requires jQuery.


****************
Testing
****************

The front-end is lacking in unit testing. There is a Jasmine based testing foundation already setup, however, Jasmine is not used thoroughly as desired. Include at least some starting tests for each JavaScript applet/project. This video of GTAC 2013 Day 2 Keynote: Testable JavaScript gives a lot of good advice in general about writing testable code, with specific examples of how to do it in JavaScript. If we wrote JavaScript the way this presentation suggests (creating clear interfaces, then testing those interfaces), writing Jasmine tests would be a lot easier.

****************
Documentation
****************

Some programmers document their code, others do not. The world of JavaScript has a nifty informal documentation spec JSDoc. It is currently at version 3. Complete reference is located at GitHub page, and official site.

As a tool, JSDoc takes JavaScript code with special ``/** */`` comments and produces HTML documentation for it. For example:

.. code-block:: javascript

    /** @namespace */
    var util = {
        /**
         * Repeat <tt>str</tt> several times.
         * @param {string} str The string to repeat.
         * @param {number} [times=1] How many times to repeat the string.
         * @returns {string}
         */
        repeat: function(str, times) {
            if (times === undefined || times < 1) {
                times = 1;
            }
            return new Array(times+1).join(str);
        }
    };

ends up looking something like: Example output by JSDoc

The following is a list of useful JSDoc (version 3) tags:

@access Specify the access level of this member - private, public, or protected.
@author Identify the author of an item.
@callback Document a callback function.
@constant Document an object as a constant.
@constructor This function is intended to be called with the "new" keyword.
@default Document the default value.
@desc Describe a symbol.
@example Provide an example of how to use a documented item.
@exports Identify the member that is exported by a JavaScript module.
@external Document an external class/namespace/module.
@file Describe a file.
@fires Describe the events this method may fire.
@global Document a global object.
@link Inline tag - create a link.
@member Document a member.
@method Describe a method or function.
@module Document a JavaScript module.
@namespace Document a namespace object.
@param Document the parameter to a function.
@private This symbol is meant to be private.
@property Document a property of an object.
@protected This member is meant to be protected.
@public This symbol is meant to be public.
@readonly This symbol is meant to be read-only.
@requires This file requires a JavaScript module.
@returns Document the return value of a function.
@see Refer to some other documentation for more information.
@summary A shorter version of the full description.
@this What does the 'this' keyword refer to here?
@throws Describe what errors could be thrown.
@todo Document tasks to be completed.
@type Document the type of an object.
@version Documents the version number of an item.

