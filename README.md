# freemarker-loader

> Render data to strings with Apache [Freemarker](http://freemarker.org)

The _`freemarker-loader`_ processes an input template with Apache Freemarker and configurable data to
generate a (raw) string that can then be further processed with [webpack](https://webpack.js.org).


## Example

```js
// Import data from data.json and render it with template.html
import html from 'html-loader!freemarker-loader?template=template.html!./data.json';
```


## Data conversion

Since Freemarker is part of the Java ecosystem, it is made for Java types, not simple JSON. For example, it
supports complex objects that follow Bean conventions, so you can use `${name}` to interpolate the result of
calling the method `getName()` of the object. Freemarker also has dedicated features to format Date objects.

To not lose all these features, the loader provides a simple mapping of JavaScript types to the Freemarker
[`TemplateModel`](http://freemarker.org/docs/api/freemarker/template/TemplateModel.html) interfaces that
extends the feature set of plain JSON.

If you want to use a Java library to deserialize the data instead, feel free to file an issue!


## Formats

Freemarker supports more output formats than just HTML. By default, the loader tries to infer the format from
the extension of the template. For example, if you load `index.html`, the loader assumes you want to use the
`html` output format. If you want a different output format, you can override that decision with the `?format`
option. The following formats are available:

- [`css`](http://freemarker.org/docs/api/freemarker/core/CSSOutputFormat.html): CSSOutputFormat
- [`html`](http://freemarker.org/docs/api/freemarker/core/HTMLOutputFormat.html): HTMLOutputFormat
- [`js`](http://freemarker.org/docs/api/freemarker/core/JavaScriptOutputFormat.html): JavaScriptOutputFormat
- [`json`](http://freemarker.org/docs/api/freemarker/core/JSONOutputFormat.html): JSONOutputFormat
- [`rtf`](http://freemarker.org/docs/api/freemarker/core/RTFOutputFormat.html): RTFOutputFormat
- [`txt`](http://freemarker.org/docs/api/freemarker/core/PlainTextOutputFormat.html): PlainTextOutputFormat
- [`xhtml`](http://freemarker.org/docs/api/freemarker/core/XHTMLOutputFormat.html): XHTMLOutputFormat
- [`xml`](http://freemarker.org/docs/api/freemarker/core/XMLOutputFormat.html): XMLOutputFormat


## Localization

Apache Freemarker offers template resolution based on the given locale. This was taken into account and can be
enabled with the `?locale` option. The `freemarker-loader` will then use the Freemarker library's resolution
mechanism to adjust the webpack resource path accordingly. You can still combine this with other loaders to
pre-process the resolved template source.

Example:

```js
// will first try index_fr_BE.html, then index_fr.html and finally index.html
import html from 'html-loader!freemarker-loader?template=index.html&locale=fr-BE!./data.json';
```


## Java Classpath and Setup

The loader supports a `?classpath` option to add items to the Java classpath. However, since it is not
predictable _when_ the loader will be called the JVM might already be initialized. To alleviate this, it is
recommended to supply any required Java options and classpath items in your webpack configuration, like so:

```js
const java = require( 'java' );
java.options.push( '-Djava.awt.headless=true' );
java.classpath.push( 'target/dependency/freemarker.jar' );

module.exports = { /* webpack config here */ };
```


## Q&A:

- **I'm getting an "unmet peer dependency" error for `java`, what up with that?**

  Since it is not possible to have multiple Java VMs in the same process (at least with the `java` node
  module), `freemarker-loader` gets out of its way to not interfere with any existing Java dependency.
  To avoid conflicts between multiple modules that need `java`, it is up to you, dear user, to provide the
  `java` module and configure it accordingly.

  As a quick remedy, simply `npm install --add java` or `yarn add java` and set it up as described above.

- **What is `java.lang.NoClassdefFoundError`?**

  You probably need to [set your classpath](#java-classpath-and-setup) correctly. The loader does not bring
  its own `freemarker.jar`, so you have to make sure you have a local copy.

- **Webpack just stops while processing the loader!**

  Due to the way the Java binding works, it might happen that it consumes all threads in the UV threadpool
  when waiting for something to be returned by the JavaScript side, especially when webpack loads multiple
  modules with this loader at the same time. Now, if the JavaScript is starting an asynchronous operation
  it will be queued indefinitely because there are no threads left.

  To work around this, you can export `UV_THREADPOOL_SIZE` to be larger than the default size "4".

