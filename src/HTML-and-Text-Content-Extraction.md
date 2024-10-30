# HTML and Text Content Extraction

### Extracting HTML and Inner HTML

Serialization enables extracting HTML content of elements, either with or without outer tags. This can be useful for accessing structured content within elements.

```rust
use dom_query::Document;

let html = r#"<!DOCTYPE html>
<html>
    <head><title>Test</title></head>
    <body><div class="content"><h1>Test Page</h1></div></body>
</html>"#;
let doc = Document::from(html);
let heading_selector = doc.select("div.content");

// Serialization including the outer HTML tag
let content = heading_selector.html();
assert_eq!(content.to_string(), r#"<div class="content"><h1>Test Page</h1></div>"#);

// Serialization excluding the outer HTML tag
let inner_content = heading_selector.inner_html();
assert_eq!(inner_content.to_string(), "<h1>Test Page</h1>");
```

The `html()` and `inner_html`() methods return serialized content as `StrTendril`. If no elements match the selector, `html()` and `inner_html()` will return an empty value, whereas `try_html()` and `try_inner_html()` return an `Option<StrTendril>`, allowing for handling of `None`.


```rust
// Using `try_html()`, which returns an Option<StrTendril>.
// If there are no matching elements, it returns None.
let opt_no_content = doc.select("div.no-content").try_html();
assert_eq!(opt_no_content, None);

// The `html()` method will return an empty `StrTendril` if there are no matches
let no_content = doc.select("div.no-content").html();
assert_eq!(no_content, "".into());

// Similarly, `inner_html()` and `try_inner_html()` work the same way
assert_eq!(doc.select("div.no-content").try_inner_html(), None);
assert_eq!(doc.select("div.no-content").inner_html(), "".into());

```

### Extracting Descendant Text

The `text()` method retrieves all descendant text content within the selected element, concatenating any nested text nodes into a single string.

```rust
use dom_query::Document;

let html = r#"<!DOCTYPE html>
<html>
    <head><title>Test</title></head>
    <body><div><h1>Test <span>Page</span></h1></div></body>
</html>"#;
let doc = Document::from(html);
let body_selection = doc.select("body div").first();
let text = body_selection.text();
assert_eq!(text.to_string(), "Test Page");

```

### Extracting Immediate Text

The `immediate_text()` method retrieves the immediate text content of the selected element, excluding any text content from its descendants.

This is useful when you need to access the text content of an element without including the text content of its child elements.


```rust
use dom_query::Document;

let html = r#"<!DOCTYPE html>
<html>
    <head><title>Test</title></head>
    <body><div><h1>Test <span>Page</span></h1></div></body>
</html>"#;

let doc = Document::from(html);

let body_selection = doc.select("body div h1").first();
// accessing immediate text without descendants
let text = body_selection.immediate_text();
assert_eq!(text.to_string(), "Test ");
```