Bundling and Minification
=========================

By `Rick Anderson`_, `Erik Reitan`_, `Daniel Roth`_

Bundling and minification are two techniques you can use in ASP.NET to improve page load performance for your web application. Bundling combines multiple files into a single file. Minification performs a variety of different code optimizations to scripts and CSS, which results in smaller payloads. Used together, bundling and minification improves load time performance by reducing the number of requests to the server and reducing the size of the requested assets (such as CSS and JavaScript files).

This article explains the benefits of using bundling and minification, including how these features can be used with ASP.NET 5 applications. 

.. contents:: In this article:
  :local:
  :depth: 1

Overview
--------

In ASP.NET Core apps, you bundle and minify the client-side resources during design-time using third party tools, such as :doc:`Gulp <using-gulp>` and :doc:`Grunt <using-grunt>`. By using design-time bundling and minification, the minified files are created prior to the application’s deployment. Bundling and minifying ahead of time provides the advantage of fewer moving parts and reduced server load. However, it’s important to recognize that design-time bundling and minification increases build complexity and only works with static files.

Bundling and minification primarily improve the first page request load time. Once a web page has been requested, the browser caches the assets (JavaScript, CSS and images) so bundling and minification won’t provide any performance boost when requesting the same page, or pages on the same site requesting the same assets. If you don’t set the expires header correctly on your assets, and you don’t use bundling and minification, the browsers freshness heuristics will mark the assets stale after a few days and the browser will require a validation request for each asset. In this case, bundling and minification provide a performance increase even after the first page request. 

Bundling
--------

Bundling is a feature that makes it easy to combine or bundle multiple files into a single file. Because bundling combines multiple files into a single file, it reduces the number of requests to the server that is required to retrieve and display a web asset, such as a web page. You can create CSS, JavaScript and other bundles. Fewer files, means fewer HTTP requests from your browser to the server or from the service providing your application. This results in improved first page load performance.

Bundling can be accomplished using the `gulp-concat <https://www.npmjs.com/package/gulp-concat>`__ task, which is installed via the Node Package Manager (`npm <https://www.npmjs.com/>`__). Add the ``gulp-concat`` package to the ``devDependencies`` section of your ``packages.json`` file:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/package.json
  :language: json
  :linenos:
  :emphasize-lines: 6

Run ``npm install`` to then install the specified packages.

.. note:: Visual Studio hides the package.json file by default. To edit package.json right-click on the npm node under Dependencies in the solution explorer and select **Open package.json**. Visual Studio will automatically install npm packages whenever package.json is modified.

In your ``gulpfile.js`` import the ``gulp-concat`` module:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/gulpfile.js
  :language: js
  :linenos:
  :lines: 4-8
  :emphasize-lines: 3

Use globbing patterns to specify the files that you want to bundle and minify:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/gulpfile.js
  :language: js
  :linenos:
  :lines: 14-19

You can then define gulp tasks that run ``concat`` on the desired files and output the result to your webroot:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/gulpfile.js
  :language: js
  :linenos:
  :lines: 31-43
  :emphasize-lines: 3, 10

Minification
------------

Minification performs a variety of different code optimizations to reduce the size of requested assets (such as CSS, image, JavaScript files). Common results of minification include removing unnecessary white space and comments, and shortening variable names to one character. 

Consider the following JavaScript function:

.. code-block:: javascript

	AddAltToImg = function (imageTagAndImageID, imageContext) {
		///<signature>
		///<summary> Adds an alt tab to the image
		// </summary>
		//<param name="imgElement" type="String">The image selector.</param>
		//<param name="ContextForImage" type="String">The image context.</param>
		///</signature>
		var imageElement = $(imageTagAndImageID, imageContext);
		imageElement.attr('alt', imageElement.attr('id').replace(/ID/, ''));
	}

After minification, the function can be reduced to the following:

.. code-block:: javascript

	AddAltToImg=function(t,a){var r=$(t,a);r.attr("alt",r.attr("id").replace(/ID/,""))};
	
In addition to removing the comments and unnecessary whitespace, the following parameters and variable names were renamed (shortened) as follows:

==================  =======  
Original            Renamed      
==================  =======  
imageTagAndImageID  t  
imageContext        a  
imageElement        r   
==================  =======  

To minify your JavaScript files you can use the `gulp-uglify <https://www.npmjs.com/package/gulp-uglify>`__ task. For CSS you can use the `gulp-cssmin <https://www.npmjs.com/package/gulp-cssmin>`__ task. Install these packages using npm as before:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/package.json
  :language: json
  :linenos:
  :emphasize-lines: 7, 8

Import the ``gulp-uglify`` and ``gulp-cssmin`` modules in your ``gulpfile.json`` file:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/gulpfile.js
  :language: js
  :linenos:
  :lines: 4-8
  :emphasize-lines: 4, 5

Run ``uglify`` to minify your bundled JavaScript files and ``cssmin`` to minify your bundled CSS files.

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/gulpfile.js
  :language: js
  :linenos:
  :lines: 31-43
  :emphasize-lines: 4, 11

To run your bundling and minification tasks you can run them from the command-line using gulp (ex ``gulp min``), or you can execute any of your gulp tasks from within Visual Studio using the Task Runner Explorer:

.. image:: bundling-and-minification/_static/task-runner-explorer.png

.. note:: The gulp tasks for bundling and minification do not general run when your project is built and must be run manually.
  
Impact of Bundling and Minification
-----------------------------------

The following table shows several important differences between listing all the assets individually and using bundling and minification on a simple web page:

==================  ==========  ============  ============  
Action              With B/M    Without B/M   Change    
==================  ==========  ============  ============  
File Requests       7           18            157%
KB Transferred      156         264.68        70%
Load Time (MS)      885         2360          167%  
==================  ==========  ============  ============  

The bytes sent had a significant reduction with bundling as browsers are fairly verbose with the HTTP headers that they apply on requests. The load time shows a big improvement, however this example was run locally. You will get greater gains in performance when using bundling and minification with assets transferred over a network. 

Controlling Bundling and Minification
-------------------------------------

In general, you want to use the bundled and minified files of your app only in a production environment. During development, you want to use your original files so your app is easier to debug. 

You can specify which scripts and CSS files to include in your pages using the environment tag helper in your layout pages(See :ref:`mvc:tag-helpers-index`). The environment tag helper will only render its contents when running in specific environments. See :doc:`/fundamentals/environments` for details on specifying the current environment.

The following environment tag will render the unprocessed CSS files when running in the ``Development`` environment:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/Views/Shared/_Layout.cshtml
  :language: html
  :linenos:
  :lines: 8-11
  :dedent: 8
  :emphasize-lines: 3

This environment tag will render the bundled and minified CSS files only when running in ``Production`` or ``Staging``:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/Views/Shared/_Layout.cshtml
  :language: html
  :linenos:
  :lines: 12-17
  :dedent: 8
  :emphasize-lines: 5

See Also
--------
- :doc:`using-gulp`
- :doc:`using-grunt`	
- :doc:`/fundamentals/environments`
- :ref:`mvc:tag-helpers-index`
	