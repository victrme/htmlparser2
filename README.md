# htmlparser2

[![NPM Version](https://img.shields.io/npm/v/htmlparser2?color=%23cb3837)](https://npmjs.org/package/htmlparser2)
[![JSR Version](https://img.shields.io/jsr/v/%40victr/htmlparser2?color=rgb(247%2C%20223%2C%2030))](https://jsr.io/@victr/htmlparser2)
[![Downloads](https://img.shields.io/npm/dm/htmlparser2.svg)](https://npmjs.org/package/htmlparser2)

The fast & forgiving HTML/XML parser, on JSR.io.

_htmlparser2 is [the fastest HTML parser](#performance), and takes some shortcuts to get there. If you need strict HTML
spec compliance, have a look at [parse5](https://github.com/inikulin/parse5)._

## Ecosystem

| Name                                                          | Description                                             |
| ------------------------------------------------------------- | ------------------------------------------------------- |
| [htmlparser2](https://github.com/fb55/htmlparser2)            | Fast & forgiving HTML/XML parser                        |
| [domhandler](https://github.com/fb55/domhandler)              | Handler for htmlparser2 that turns documents into a DOM |
| [domutils](https://github.com/fb55/domutils)                  | Utilities for working with domhandler's DOM             |
| [css-select](https://github.com/fb55/css-select)              | CSS selector engine, compatible with domhandler's DOM   |
| [cheerio](https://github.com/cheeriojs/cheerio)               | The jQuery API for domhandler's DOM                     |
| [dom-serializer](https://github.com/cheeriojs/dom-serializer) | Serializer for domhandler's DOM                         |

## Usage

`htmlparser2` itself provides a callback interface that allows consumption of documents with minimal allocations. For a
more ergonomic experience, read [Getting a DOM](#getting-a-dom) below.

```js
import * as htmlparser2 from "@victr/htmlparser2";

const parser = new htmlparser2.Parser({
  onopentag(name, attributes) {
    /*
     * This fires when a new tag is opened.
     *
     * If you don't need an aggregated `attributes` object,
     * have a look at the `onopentagname` and `onattribute` events.
     */
    if (
      name === "script" && attributes.type === "text/javascript"
    ) {
      console.log("JS! Hooray!");
    }
  },
  ontext(text) {
    /*
     * Fires whenever a section of text was processed.
     *
     * Note that this can fire at any point within text and you might
     * have to stitch together multiple pieces.
     */
    console.log("-->", text);
  },
  onclosetag(tagname) {
    /*
     * Fires when a tag is closed.
     *
     * You can rely on this event only firing when you have received an
     * equivalent opening tag before. Closing tags without corresponding
     * opening tags will be ignored.
     */
    if (tagname === "script") {
      console.log("That's it?!");
    }
  },
});
parser.write(
  "Xyz <script type='text/javascript'>const foo = '<<bar>>';</script>",
);
parser.end();
```

Output (with multiple text events combined):

```
--> Xyz
JS! Hooray!
--> const foo = '<<bar>>';
That's it?!
```

This example only shows three of the possible events. Read more about the parser, its events and options in the
[wiki](https://github.com/fb55/htmlparser2/wiki/Parser-options).

## Getting a DOM

The `DomHandler` produces a DOM (document object model) that can be manipulated using the
[`DomUtils`](https://github.com/fb55/DomUtils) helper.

```js
import * as htmlparser2 from "@victr/htmlparser2";

const dom = htmlparser2.parseDocument(htmlString);
```

The `DomHandler`, while still bundled with this module, was moved to its
[own module](https://github.com/fb55/domhandler). Have a look at that for further information.

## Parsing Feeds

`htmlparser2` makes it easy to parse RSS, RDF and Atom feeds, by providing a `parseFeed` method:

```javascript
const feed = htmlparser2.parseFeed(content, options);
```

## Performance

After having some artificial benchmarks for some time, **@AndreasMadsen** published his
[`htmlparser-benchmark`](https://github.com/AndreasMadsen/htmlparser-benchmark), which benchmarks HTML parses based on
real-world websites.

At the time of writing, the latest versions of all supported parsers show the following performance characteristics on
GitHub Actions (sourced from
[here](https://github.com/AndreasMadsen/htmlparser-benchmark/blob/e78cd8fc6c2adac08deedd4f274c33537451186b/stats.txt)):

```
htmlparser2        : 2.17215 ms/file ± 3.81587
node-html-parser   : 2.35983 ms/file ± 1.54487
html5parser        : 2.43468 ms/file ± 2.81501
neutron-html5parser: 2.61356 ms/file ± 1.70324
htmlparser2-dom    : 3.09034 ms/file ± 4.77033
html-dom-parser    : 3.56804 ms/file ± 5.15621
libxmljs           : 4.07490 ms/file ± 2.99869
htmljs-parser      : 6.15812 ms/file ± 7.52497
parse5             : 9.70406 ms/file ± 6.74872
htmlparser         : 15.0596 ms/file ± 89.0826
html-parser        : 28.6282 ms/file ± 22.6652
saxes              : 45.7921 ms/file ± 128.691
html5              : 120.844 ms/file ± 153.944
```