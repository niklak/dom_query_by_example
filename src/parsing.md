# Parsing

### Parsing HTML Documents

The `Document` struct in `dom_query` is designed to handle full HTML documents. You can create a `Document` by passing in HTML content, which can be provided in several formats: `&str`, `String`, or `StrTendril`.

```rust
extern crate dom_query;

use dom_query::Document;
use tendril::StrTendril;

// HTML content as a string slice
let contents_str = r#"<!DOCTYPE html>
<html><head><title>Test Page</title></head><body></body></html>"#;
let doc = Document::from(contents_str);

// HTML content as a String
let contents_string = contents_str.to_string();
let doc = Document::from(contents_string);

// HTML content as a StrTendril
let contents_tendril = StrTendril::from(contents_str);
let doc = Document::from(contents_tendril);

// Checking the root element of the `Document`
assert!(doc.root().is_document());
```

When parsing a full HTML document, `Document` will recognize a `<!DOCTYPE>` if it exists at the start of the input. In this case, the `Doctype` will be added as the first child of the root `Document` node. If you provide an HTML snippet without a `<!DOCTYPE>`, `Document` will ignore the Doctype.

```rust
assert!(doc.root().first_child().unwrap().is_doctype());
```


### Parsing HTML Fragments

For cases where you need to parse only a part of an HTML document, such as a snippet or component, `dom_query` provides` Document::fragment()`. This function also accepts `&str`, `String`, or `StrTendril`, but behaves a little differently from `Document::from()` in that it treats the input as a fragment instead of a full document.


```rust
use dom_query::Document;
use tendril::StrTendril;

// Parsing an HTML fragment from a string slice
let contents_str = r#"<div><p>Example Fragment</p></div>"#;
let fragment = Document::fragment(contents_str);

// Parsing from a String
let contents_string = contents_str.to_string();
let fragment = Document::fragment(contents_string);

// Parsing from a StrTendril
let contents_tendril = StrTendril::from(contents_str);
let fragment = Document::fragment(contents_tendril);

// Checking the root element of the fragment
assert!(!fragment.root().is_document());
assert!(fragment.root().is_fragment());
```

When using `Document::fragment()`, note that Doctype declarations are ignored, focusing only on the fragment itself.


```rust
// Confirming Doctype is excluded in the fragment
assert!(!fragment.root().first_child().unwrap().is_doctype());
```
`Document::fragment()` is also used internally within the library to create new elements within the document tree.
