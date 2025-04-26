## Serializing Document to Markdown

With the `markdown` feature enabled, you can serialize a `Document` or `NodeRef` into Markdown format using the `md` method.


```rust
use dom_query::Document;

let contents = "
<style>p {color: blue;}</style>
<p>I really like using <b>Markdown</b>.</p>

<p>I think I'll use it to format all of my documents from now on.</p>";

let expected = "I really like using **Markdown**\\.\n\n\
I think I'll use it to format all of my documents from now on\\.";

let doc = Document::from(contents);

// Passing `None` into `md` will apply the default list of skipped tags: 
// `["script", "style", "meta", "head"]`.
let got = doc.md(None);

assert_eq!(got.as_ref(), expected);

// If you want to serialize the entire document without skipping any elements,
// pass `Some(&vec![])` into `md`.
```

When using `Document::from`, be aware that `html5ever` automatically constructs an HTML `<head>` element if necessary and may move `<style>`, `<meta>`, and similar tags into it.
If you want to preserve the original content order (as provided), it's recommended to use `Document::fragment`.

```rust
let contents = "<style>p {color: blue;}</style>\
<div><h1>Content Heading</h1></div>\
<p>I really like using Markdown.</p>\
<p>I think I'll use it to format all of my documents from now on.</p>";

let expected = "p \\{color: blue;\\}\n\
I really like using Markdown\\.\n\n\
I think I'll use it to format all of my documents from now on\\.";

let doc = Document::fragment(contents);

// Here we tell `md` to skip `<div>` elements during serialization.
let got = doc.md(Some(&["div"]));

assert_eq!(got.as_ref(), expected);
```

### Notes:

- Skipped elements are completely ignored in the output.
- Backslashes (`\`) are inserted automatically to escape special Markdown characters when needed.