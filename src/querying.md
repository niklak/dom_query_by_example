## Querying

### Selecting Elements

The `dom_query` crate provides several selection methods to locate HTML elements in the document. Using CSS-like selectors, you can select both single and multiple elements.

```rust
use dom_query::Document;

let html = r#"<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Test Page</title>
    </head>
    <body>
        <h1>Test Page</h1>
        <ul>
            <li>One</li>
            <li><a href="/2">Two</a></li>
            <li><a href="/3">Three</a></li>
        </ul>
    </body>
</html>"#;
let document = Document::from(html);

// Select a single element
let a = document.select("ul li:nth-child(2)");
let text = a.text().to_string();
assert!(text == "Two");

// Selecting multiple elements
document.select("ul > li:has(a)").iter().for_each(|sel| {
    assert!(sel.is("li"));
});

// Optionally select an element with `try_select`, which returns an `Option`
let no_sel = document.try_select("p");
assert!(no_sel.is_none());
```

The `Selection::is` method checks whether elements in the current selection match a given selector, without performing a deep search within the elements.
`dom_query` supports pseudo-classes that goes from `selectors` crate and a few others from itself.

**See also**: [List of supported CSS pseudo-classes](./List-of-supported-CSS-pseudo‐classes.md)


### Selecting a Single Match and Multiple Matches

To retrieve only the first match of a selector, `Selection::select_single` method is available. This method is useful when you want a single match without iterating through all matches.


```rust
use dom_query::Document;

let doc: Document = r#"<!DOCTYPE html>
<html lang="en">
<head></head>
<body>
    <ul class="list">
        <li>1</li><li>2</li><li>3</li>
    </ul>
    <ul class="list">
        <li>4</li><li>5</li><li>6</li>
    </ul>
</body>
</html>"#.into();

// selecting a first match
let single_selection = doc.select_single(".list");
assert_eq!(single_selection.length(), 1);
assert_eq!(single_selection.inner_html().to_string().trim(), 
    "<li>1</li><li>2</li><li>3</li>");

// selecting all matches
let selection = doc.select(".list");
assert_eq!(selection.length(), 2);
// but when you call property methods usually
// you will get the result of the first match
assert_eq!(selection.inner_html().to_string().trim(), 
    "<li>1</li><li>2</li><li>3</li>");

// This creates a Selection from the first node in the selection
let first_selection = doc.select(".list").first();
assert_eq!(first_selection.length(), 1);
assert_eq!(first_selection.inner_html().to_string().trim(), 
    "<li>1</li><li>2</li><li>3</li>");

// This approach also creates a new Selection from the next node, each iteration
let next_selection = doc.select(".list").iter().next().unwrap();
assert_eq!(next_selection.length(), 1);
assert_eq!(next_selection.inner_html().to_string().trim(), 
    "<li>1</li><li>2</li><li>3</li>");

// currently, to get data from all matches you need to iterate over them:
let all_matched: String = selection
.iter()
.map(|s| s.inner_html().trim().to_string())
.collect();

assert_eq!(
    all_matched,
    "<li>1</li><li>2</li><li>3</li><li>4</li><li>5</li><li>6</li>"
);

// same thing as previous, but a little cheaper, because we iterating over the nodes, 
// and do not create a new Selection on each iteration
let all_matched: String = doc
        .select(".list").nodes()
        .iter()
        .map(|s| s.inner_html().trim().to_string())
        .collect();

assert_eq!(
    all_matched,
    "<li>1</li><li>2</li><li>3</li><li>4</li><li>5</li><li>6</li>"
);

```

### Descendant selections

Elements can be selected in relation to a parent element. Here, a `Document` is queried for ul elements, and then descendant selectors are applied within that context.

```rust
use dom_query::Document;

let html = r#"<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Test Page</title>
    </head>
    <body>
        <h1>Test Page</h1>
        <ul class="list-a">
            <li>One</li>
            <li><a href="/2">Two</a></li>
            <li><a href="/3">Three</a></li>
        </ul>
        <ul class="list-b">
            <li><a href="/4">Four</a></li>
        </ul>
    </body>
</html>"#;
let document = Document::from(html);

// selecting parent elements
let ul = document.select("ul");
ul.select("li").iter().for_each(|el| {
    // descendant select matches only inside the children context
    assert!(el.is("li"));
});

// also descendant selector may include elements of the higher level than the parent. 
// It may be useful to specify the exact element you want to select
let el = ul.select("body ul.list-b li").first();
let text = el.text();
assert_eq!("Four", text.to_string());
```

### Selecting with Precompiled Matchers

For repeated queries, `dom_query` allows using precompiled matchers. This approach enhances performance when matching the same pattern across multiple documents.

```rust
use dom_query::{Document, Matcher};

let html1 = r#"<!DOCTYPE html>
    <html><head><title>Test Page 1</title></head><body></body></html>"#;
let html2 = r#"<!DOCTYPE html>
    <html><head><title>Test Page 2</title></head><body></body></html>"#;
let doc1 = Document::from(html1);
let doc2 = Document::from(html2);

// create a matcher once, reuse on different documents
let title_matcher = Matcher::new("title").unwrap();

let title_el1 = doc1.select_matcher(&title_matcher);
assert_eq!(title_el1.text(), "Test Page 1".into());

let title_el2 = doc2.select_matcher(&title_matcher);
assert_eq!(title_el2.text(), "Test Page 2".into());

let title_single = doc1.select_single_matcher(&title_matcher);
assert_eq!(title_single.text(), "Test Page 1".into());
```

### Selecting Ancestor Elements

You can use `Node::ancestors()` to retrieve the sequence of ancestor nodes for a given element in the document tree, which can be helpful when you need to navigate upward from a specific node.

```rust

use dom_query::Document;

let doc: Document = r#"<!DOCTYPE html>
<html>
    <head>Test</head>
    <body>
        <div id="great-ancestor">
            <div id="grand-parent">
                <div id="parent">
                    <div id="child">Child</div>
                </div>
            </div>
        </div>
    </body>
</html>
"#.into();

// Select an element
let child_sel = doc.select("#child");
assert!(child_sel.exists());

// Access the selected node
let child_node = child_sel.nodes().first().unwrap();

// Get all ancestor nodes for the `#child` node
let ancestors = child_node.ancestors(None);
let ancestor_sel = Selection::from(ancestors);

// In this case, all ancestor nodes up to the root <html> are included
assert!(ancestor_sel.is("html")); // Root <html> is included
assert!(ancestor_sel.is("#parent")); // Direct parent is also included

// `Selection::is` performs a shallow match, so it will not match `#child` in this selection.
assert!(!ancestor_sel.is("#child"));

// You can limit the number of ancestor nodes returned by specifying `max_limit`
let limited_ancestors = child_node.ancestors(Some(2));
let limited_ancestor_sel = Selection::from(limited_ancestors);

// With a limit of 2, only `#grand-parent` and `#parent` ancestors are included
assert!(limited_ancestor_sel.is("#grand-parent"));
assert!(limited_ancestor_sel.is("#parent"));
assert!(!limited_ancestor_sel.is("#great-ancestor")); // This node is excluded due to the limit
```

### Selecting with pseudo-classes (:has, :has-text, :contains)

The `dom_query` crate provides versatile selector pseudo-classes, built on both its own functionality and the capabilities of the `selectors` crate. These pseudo-classes allow targeting elements based on attributes, text content, and context within the document.

```rust
use dom_query::Document;

let html = include_str!("../test-pages/rustwiki_2024.html");
let doc = Document::from(html);

// Search for list items (`li`) within a `tr` element that contains an `a` element
// with the title "Programming paradigm"
let paradigm_selection = doc.select(
    r#"table tr:has(a[title="Programming paradigm"]) td.infobox-data ul > li"#
    );

println!("Rust programming paradigms:");
for item in paradigm_selection.iter() {
    println!(" {}", item.text());
}
println!("{:-<50}", "");

// Select items based on `th` containing text "Influenced by" and
// the following `tr` containing `td` with list items.
let influenced_by_selection = doc.select(
    r#"table tr:has-text("Influenced by") + tr td ul > li > a"#
    );

println!("Rust influenced by:");
for item in influenced_by_selection.iter() {
    println!(" {}", item.text());
}
println!("{:-<50}", "");

// Extract all links within a paragraph containing "foreign function interface" text.
// Since a part of the text is in a separate tag, we use the `:contains` pseudo-class.
let links_selection = doc.select(
    r#"p:contains("Rust has a foreign function interface") a[href^="/"]"#
    );

println!("Links in the FFI block:");
for item in links_selection.iter() {
    println!(" {}", item.attr("href").unwrap());
}
println!("{:-<50}", "");

// :only-text selects an element that contains only a single text node, 
// with no child elements.
// It can be combined with other pseudo-classes to achieve more specific selections.
// For example, to select a <div> inside an <a> 
//that has no siblings and no child elements other than text.
println!("Single <div> inside an <a> with text only:");
for el in doc.select("a div:only-text:only-child").iter() {
    println!("{}", el.text().trim());
}

```
#### Key Points:

- `:has(selector)`: Finds elements that contain a matching element anywhere within.
- `:has-text("text")`: Matches elements based on their immediate text content, ignoring any nested elements. This makes it ideal for selecting nodes where the direct text is crucial for differentiation.
- `:contains("text")`: Selects elements containing the specified text within them, useful when searching in a block of text.
- `:only-text`: Selects elements that contain only a single text node, with no other child nodes.

These pseudo-classes allow for precise and expressive searches within the DOM, enabling the selection of content-rich elements based on structural or attribute-driven conditions.
For a full list of supported pseudo-classes, refer to the  [Supported CSS Pseudo-Classes List](./supported-css-pseudo-classes.md).