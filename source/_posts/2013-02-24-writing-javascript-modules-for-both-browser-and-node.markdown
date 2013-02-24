---
layout: post
title: "Writing JavaScript modules for both Browser and Node.js"
date: 2013-02-24 11:04
comments: true
categories: [Development, JavaScript]
---

I recently started the complete refactor of [feathe.rs](http://feathe.rs "Feathers, blogless writing since 2012") and immediately faced with the issue of code reuse between client and server. The app is completely written in JavaScript using Node.js on the server side and requires several validation routines that should also run on the client Browser. With this short article I'll explain how to write modules whose code can be easily reused in both Browser and Node.js.

<!-- more-->

Let's assume we want to create a module with a `Validator` object that exports a serie of routines for validating stuff. The Node.js approach would be creating a `validator.js` file with the following content:

{% codeblock lang:javascript %}
var Validator = (function() {
  var Validator = function(options) {
    ...
  };

  Validator.prototype.foo = function foo() {
    ...
  };

  return Validator;
})();

module.exports = Validator;
{% endcodeblock %}

and then access it in using the `require` function:

{% codeblock lang:javascript %}
var Validator = require('./validator');
var v = new Validator({...});
v.foo(...);
{% endcodeblock %}

To use it in the client we could do the following:

{% codeblock lang:html %}
<script src="validator.js"></script>
<script>
  var v = new Validator({...});
  v.foo(...);
</script>
{% endcodeblock %}

The above code will throw an error as the `module` variable used in `validator.js` is not defined. However the `Validator` object sill gets exported, contrary to what happens in Node.js where only what's actually being assigned to `module.exports` is visible outside the module. In order to fix this weird behaviour we have first to deal with the `module` definition and then with information hiding.

The workaround is to check for `module` variable definition (including `exports`) and if undefined, as in the Browser, associate whatever gets exported to the `window` scope. In addition to it, as to keep information hiding all module's code should be wrapped into a closure. The resulting module's code would look like this:

{% codeblock lang:javascript %}

(function() {
  var Validator = (function() {
    var Validator = function(options) {
      ...
    };

    Validator.prototype.foo = function foo() {
      ...
    };

    return Validator;
  })();

  if (typeof module !== 'undefined' && typeof module.exports !== 'undefined')
    module.exports = Validator;
  else
    window.Validator = Validator;
})();

{% endcodeblock %}

If you want to support `AMD` you can modify the exporting block as follows:

{% codeblock lang:javascript %}

  ...

  if (typeof module !== 'undefined' && typeof module.exports !== 'undefined') {
    module.exports = Validator;
  }
  else {
    if (typeof define === 'function' && define.amd) {
      define([], function() {
        return Validator;
      });
    }
    else {
      window.Validator = Validator;
    }
  }

  ...

{% endcodeblock %}