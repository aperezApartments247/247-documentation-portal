# 247 JavaScript Standards and Conventions

jQuery is available on all pages of the CMS and should always be
available on rendered sites as well. We do not invoke jQuery in
`noConflict` mode, so any other libraries you bring in need to not
override the `$`. We try to always be on the newest version of jQuery,
too.

Javascript should conform to the [jQuery style
guide](http://contribute.jquery.org/style-guide/js/) with a few
exceptions.

### Linting

The style guide requires certain rules in the JSHint configuration. We
amend those rules a little bit. We still have clients that are locked
into using IE8, and as such we need to be careful to not deliberately
break old browsers. So the `es3` option is very handy. Also, because
we're a python shop we indent code with spaces in place of tabs. So the
`smarttabs` option is unneeded in our code base, but checking for four
space indentation is quite nice. That leaves the JSHint configuration
with any project specific stuff plus these required rules:

```javascript
{
    "boss": true,
    "curly": true,
    "eqeqeq": true,
    "eqnull": true,
    "es3": true,
    "expr": true,
    "immed": true,
    "indent": 4,
    "noarg": true,
    "onevar": true,
    "quotmark": "double",
    "trailing": true,
    "undef": true,
    "unused": true
}
```

### Spacing

As I mentioned before, we are a python shop and the python style guide
dictates we indent code with spaces instead of tabs. In order to avoid
having to maintain different editor configurations based on languages,
javascript code in our repository should also use spaces for code
indentation.

Additionally, we do not place extra spaces around arguments or elements
when calling functions or declaring objects. For example:

```javascript
// Bad
array = [ "*" ];
array = [ a, b ];

foo( arg );
foo( "string", object );
foo( options, object[ property ] );
foo( node, "property", 2 );

// Good
array = ["*"];
array = [a, b];

foo(arg);
foo("string", object);
foo(options, object[property]);
foo(node, "property", 2);
```

### Type Checks

The jQuery style guide is fairly reasonable with regard to type checking,
but to gaurd against any possible issues with jQuery changes, we try to
stick with native javascript type checks wherever they are applicaple,
rather than using jQuery methods. One thing to make note of however is
checking for an `Array` type. If the script will not be working across
frames or windows, the instanceof operator will suffice. Otherwise a
library method (whether from jQuery, underscore, or others) is warrented.
For example:

```javascript
// String
typeof object === "string";

// Number
typeof object === "number";

// Boolean
typeof object === "boolean";

// Object
typeof object === "object";

// Function
typeof object === "function";

// Array
object instanceof Array;
// Or if they array might be in another frame or window
$.isArray(object);

// Element
object.nodeType

// Null
object === null;

// Null or undefined
object == null;

// Undefined global variable
typeof object === "undefined";

// Undefined local variable
object === undefined;

// Undefined object properties
object.prop === undefined;
```

### Quotes

The jQuery style guide is pretty straight forward with regard to quotes,
and we generally hold to that. We only use single quotes for strings when
there needs to be a double quote in the string. In that event we prefer to
use single quotes for the string instead of escaping quotes. Also, when we
are quoting markup (which should be avoided when possible, but is sometimes
necessary) that needs quoted attributes, we use double quotes for the
attributes and single quotes for the strings.

```javascript
var normalString = "This is 'normal' string content.",
    stringWithQuotes = 'This string needs "double" quotes.',
    htmlString = "<p>HTML with no attributes</p>",
    htmlAttributes = '<p id="awesome_paragraph">HTML with attributes</p>';
```

### Full file closures

The jQuery style guide rules that full file closures should not receive
indentation on their function body, and that AMD/UMD module definitions
should receive that same no-indentation treatment. In our code, we prefer
to go ahead and indent in those situations just as you would any other
function. This avoids any inconsistencies on when indentation should or
should not be used, and makes files easier to read.

### Compiled JS CMS App and using eslint (not using React)

For those who use `eslint` and are compiling an app not using React for the CMS...

Here is your .eslintrc.js:

```js
module.exports = {
      "env": {
          "browser": true,
          "es6": true
      },
      "extends": "airbnb-base",
      "rules": {
          "indent": ["error", 4],
          "linebreak-style": [
              "error",
              "unix"
          ],
          "quotes": ["error", "double"],
          "prefer-arrow-callback": 0,
          "func-names": ["warn", "as-needed"],
          "semi": [
              "error",
              "always"
          ],
          "comma-dangle": ["error", "never"],
      },
  };
```

Last tested with:
```
  "devDependencies": {
    // ...
    "eslint": "^4.19.1",
    "eslint-config-airbnb-base": "^13.1.0",
    "eslint-plugin-import": "^2.14.0",
    // ...
  },
```

### React

React Apps should conform to the [React style guide](https://github.com/airbnb/javascript/tree/master/react#airbnb-reactjsx-style-guide) and should use [ESLint](https://eslint.org/) instead of [JSHint](http://jshint.com/) with the following .eslintrc in the project root

```
{
    "extends": "airbnb",
    "env": {
        "browser": true,
    },
    "rules": {
        "quotes": [2, "double"],
        "indent": ["error", 4],
        "react/jsx-filename-extension": [1, {
            "extensions": [".js", ".jsx"]
        }],
        "react/jsx-indent": [2, 4],
        "react/jsx-indent-props": [2, 4],
        "react/no-did-mount-set-state": [0],
    }
}
```

This config is known to work with the following versions of npm modules:
```
eslint@5.1.0
eslint-plugin-import@2.13.0
eslint-plugin-react@7.10.0
eslint-plugin-jsx-a11y@6.1.1
eslint-config-airbnb@17.0.0
```