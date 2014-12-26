Django React
============

Render and bundle React components from a Django application.

Documentation
-------------

- [Basic usage](#basic-usage)
- [Installation](#installation)
- [Running the tests](#running-the-tests)

Basic usage
-----------

Define your component

```python
from django_react import ReactComponent

class MyComponent(ReactComponent):
    source = 'path/to/file.jsx'
```

Create an instance of your component and pass it some props

```python
my_component = MyComponent(
    some_prop='foo',
    some_other_prop=[1, 2, 3]
)
```

In your template you can now render the component and inject your
component's source.

```html
{{ my_component.render_to_string }}

<script src="path/to/react.js"></script>

{{ my_component.render_js }}
```

The rendered JavaScript will automatically include:
- Your props, serialized to JSON
- Your source, which will have been JSX transformed and bundled with Webpack
- Initialization code that immediately mounts your component with React

The user will see the rendered component immediately and React will automatically
start to add interactivity as the page loads the JavaScript.

Dependencies
------------

- Node.js >= 0.10.0
- NPM >= 1.2.0

Installation
------------

```bash
pip install django-react
```

Add `'django_react'` to your `INSTALLED_APPS` setting

```python
INSTALLED_APPS = (
    # ...
    'django_react',
)
```

Note: You may experience a short, once-off pause when you first attempt to boot Django.
The pause is caused by NPM installing the JavaScript libraries which Django React
requires to operate.

ReactComponent
--------------

A class which integrates server-side rendering of react components with Webpack's bundling.

### ReactComponent.render_to_string()

Render a component to its initial HTML. You can use this method to generate HTML
on the server and send the markup down on the initial request for faster page loads
and to allow search engines to crawl you pages for SEO purposes.

`render_to_string` takes an optional argument `wrap` which, if set to False, will
prevent the the rendered output from being wrapped in the container element
generated by the component's `render_container` method.

Note that the rendered HTML will be wrapped in a container element generated by the component's
`render_container` method.

```html
{{ component.render_to_string }}
```

### ReactComponent.render_to_static_markup()

Similar to `ReactComponent.render_to_string`, except this doesn't create
extra DOM attributes such as `data-react-id`, that React uses internally.
This is useful if you want to use React as a simple static page generator,
as stripping away the extra attributes can save lots of bytes.

```html
{{ component.render_to_static_markup }}
```

### ReactComponent.render_js()

Renders the script elements containing the component's props, source and
initialisation.

`render_js` is effectively shorthand for calling each of the component's
`render_props`, `render_source`, and `render_init` methods.

```html
{{ component.render_js }}
```

### ReactComponent.render_container()

Renders a HTML element with attributes which Django React uses internally to
facilitate mounting components with React.

The rendered element is provided with a id attribute which is generated by the component's
`get_container_id` method.

`render_container` takes an optional argument, `content` which should be a string to be
inserted within the element.

```html
{{ component.render_container }}

<script>
    console.log(document.getElementById('{{ component.get_container_id }}'));
</script>
```

### ReactComponent.render_props()
### ReactComponent.render_source()
### ReactComponent.render_init()

ReactBundle
-----------

A `django_webpack.WebpackBundle` which is configured to:
- support loading JSX files
- omit React from the generated bundle

You can extend the Webpack configuration by inheriting from ReactBundle and assigning
the class as the `bundle` attribute on your ReactComponent subclass.

```python
from django_react import ReactBundle, ReactComponent

class JqueryOptimisedReactBundle(ReactBundle):
    no_parse = ('jquery',)

class MyComponent(ReactComponent):
    bundle = JqueryOptimisedReactBundle
```

render_component()
--------------

Render a component to its initial HTML. You can use this method to generate HTML
on the server and send the markup down on the initial request for faster page loads
and to allow search engines to crawl you pages for SEO purposes.

Arguments:

- `path_to_source`: an absolute path to a JS or JSX file which exports the component.
- `serialized_props`: [optional] a string containing the JSON-serialized props which will
  be passed to the component.
- `to_static_markup`: [optional] a boolean indicating that React's `render_to_static_markup`
  method should be used for the rendering. Set this to True if you are rendering the component
  to static HTML and you have no need for client-side use of React.

```python
import json
from django_react import render_component

props = {
    'foo': 1,
    'bar': [1, 2, 3],
}

html = render_component(
    path_to_source='/path/to/component.jsx',
    serialized_props=json.dumps(props),
)
```

Running the tests
-----------------

```bash
mkvirtualenv django-react
pip install -r requirements.txt
python django_react/tests/runner.py
```
