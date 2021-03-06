# snabbdom-template

Content insertion is accomplished by targeting standard HTML tag names and other CSS selectors.
Works with [Snabbdom](https://github.com/paldepind/snabbdom).

This module allows you to develop plain HTML/CSS templates with dummy data in the places dynamic
data will be inserted. The benefit is that the templates can be developed independently from the
programming that dynamically incorporates data into the web page.

### layering templates and map example

```
var outer = fs.readFileSync('public/outer.html', 'utf-8')
var section = fs.readFileSync('public/section.html', 'utf-8')
var template = fs.readFileSync('public/template.html', 'utf-8')

var vtree = snabbdomTemplate([outer, section, template], {
  '#sectionheader': 'Start Here:',
  'div#message': {class: 'myclass', '_html': 'Ready.'},
  '#mapme': {_map: {'li': ['one', 'two', 'three', 'four']}},
})
```

outer.html

```
<!DOCTYPE html>
<html>
  <head>
    <title>snabbdom-template</title>
    <link rel="stylesheet" type="text/css" href="public/css/main.css">
  </head>
  <body>
    <div class="template">template</div>
    <div id="scripts">
      <script src="public/js/main.js"></script>
    </div>
  </body>
</html>
```

section.html

```
      <h2 id="sectionheader">section header</h2>
      <div class="template"></div>
```

template.html

```
        <div><div id="message">message goes here</div>
        <ul id="mapme">
          <li>a</li>
          <li>b</li>
          <li>c</li>
        </ul>
```

...outputs a vtree that renders to:

```
<!DOCTYPE html>
<html>
  <head>
    <title>snabbdom-template</title>
    <link rel="stylesheet" type="text/css" href="public/css/main.css">
  </head>
  <body>
  </head>
  <body>
    <div class="template">
      <h2 id="sectionheader">Start Here:</h2>
      <div class="template">
        <div><div id="message" class="myclass">Ready.</div>
        <ul id="mapme">
          <li>one</li>
          <li>two</li>
          <li>three</li>
          <li>four</li>
        </ul>
      </div>
    </div>
    <div id="scripts">
      <script src="public/js/main.js"></script>
    </div>
  </body>
</html>
```

### vtree example

```
var sh = require('snabbdom/h')
var outer = fs.readFileSync('public/outer.html', 'utf-8')

var vtree = snabbdomTemplate(outer, sh('div.myclass', 'My content.'))
```

...outputs a vtree that renders to:

```
<!DOCTYPE html>
<html>
  <head>
    <title>snabbdom-template</title>
    <link rel="stylesheet" type="text/css" href="public/css/main.css">
  </head>
  <body>
  </head>
  <body>
    <div class="template">
      <div class="myclass">My content.</div>
    </div>
    <div id="scripts">
      <script src="public/js/main.js"></script>
    </div>
  </body>
</html>
```

[main-loop](https://www.npmjs.com/package/main-loop) example
```
var fs = require('fs')
var sT = require('snabbdom-template')
var mL = require('main-loop')

var patch = require('snabbdom').init([
  require('snabbdom/modules/eventlisteners').default,
])
var initstate = {n:0}

var outer = fs.readFileSync('public/mainsimple.html', 'utf-8')
var inner = fs.readFileSync('public/button.html', 'utf-8')

var view = function (state) {
  return sT([outer, inner], {
    '#count': state.n,
    'button': {'_on': {click: onclick}}
  })
}

var loop = mL(initstate, view, {
  target: document.getElementById('content'),
  diff: function (tree, newTree) { return newTree }, // snabbdom incorporates diff in patch
  patch: patch
})
loop.update(initstate)

function onclick () {
  initstate.n++
  loop.update(initstate)
}
```
...then `browserify -t brfs index.js > public/bundle.js` and include `<script src="bundle.js"></script>` at the bottom of your HTML body.

### license

MIT
