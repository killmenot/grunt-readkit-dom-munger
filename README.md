# grunt-readkit-dom-munger

> Read and manipulate HTML documents with cheerio.

This task is a patched version of [grunt-dom-munger](https://github.com/cgross/grunt-dom-munger), created in order to support [Readk.it](http://readk.it) requirements:

## Patched code
Specifically, the patched code is as follows:

### dom_munger.js

In the function ```processFile``` in ```node_modules/grunt-dom-munger/tasks/dom_munger.js``` change:

      updatedContents = $.html();  

to 

      if (options.xmlMode) {
        updatedContents = $.xml();
      } else {
        updatedContents = $.html();  
      }

We also add the following code to allow us to specify a regExp to identify a string to be replaced with a supplied value

        if (options.read.replace) {
          var regexp = new RegExp(options.read.replace);
          vals = vals.map(function(val){
            return val.replace(regexp, options.read.replacewith);
          });
        }

### Parser.js

In ```node_modules/grunt-dom-munger/node_modules/cheerio/node_modules/htmlparser2/lib/Parser.js``` comment out the meta reference in ```voidElements```, otherwise we end up with broken meta tags in the opf file (specifically, meta tags that have both an opening and a closing tag lose their closing tag):

      var voidElements = {
      ...
      //  meta: true,
      ...
      };

### jsDom

We remove jsDOM, as this is an optional dependency and not needed for Readk.it purposes.

## Getting Started
This plugin requires Grunt `~0.4.1`

```shell
npm install grunt-readkit-dom-munger --save-dev
```

One the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-readkit-dom-munger');
```

## The "readkit_dom_munger" task

### Overview
The readkit-dom-munger reads one or more HTML files and performs one or more operations on them.  

```js
grunt.initConfig({
  readkit_dom_munger: {
    index_reader: {
      // Update our index.html to point to the built readkit,
      // remove unnecessary data attributes, and add tags for bake.
      options: {
        callback: function($) {
          $('.readkit-library').removeAttr('data-library');
          $('#readkit-entry').removeAttr('data-main');
          $('link[rel="stylesheet"]').remove();
          $('link[rel="apple-touch-icon-precomposed"]').remove();
          $('link[rel="apple-touch-startup-image"]').remove();
          $('link[rel="shortcut icon"]').remove();
          $('link[rel="stylesheet"][href="fonts/fontello/css/fontello.css"]').remove();
          $('meta[name="apple-mobile-web-app-capable"]').remove();
          $('meta[name="apple-mobile-web-app-status-bar-style"]').remove();
          $('head').append('<style><!--(bake css/screen.css)--></style>');
          $('script#readkit-client').removeAttr('src').append('<!--(bake js/client.config.js)-->');
          $('script#readkit-entry').removeAttr('src').append('<!--(bake ../readkit.js)-->');
        }
      },
      src: ['build/readkit/index.html']
    },
    index_library: {
      // Update our index.html to point to the built readkit,
      // and remove unnecessary data attributes.
      options: {
        update: {selector: '#readkit-entry', attribute: 'src', value: 'js/readkit.js'},
        callback: function($) {
          $('#readkit-client').attr('src', 'library/js/library.client.config.js');
          $('#readkit-entry').removeAttr('data-main');
        }
      },
      src: ['dist/readkit.library/index.html']
    },
    library: {
      // Remove unnecessary data attributes.
      options: {
        update: {selector: '#library-entry', attribute: 'src', value: 'js/main.compiled.js'},
        callback: function($) {
          $('#library-entry').removeAttr('data-main');
        }
      },
      src: ['dist/readkit.library/library/library.html']
    }
    // Others set dynamically
  },
})
```

For more information, consult the [grunt-dom-munger](https://github.com/cgross/grunt-dom-munger) documentation.

## Release History

 * v2.0.0 - Initial release (using grunt_dom_munger v2.0.0)
