# Manipulating the DOM

## Manipulating the Selection
The `dom_query` crate provides various methods to manipulate the DOM. Below are some examples demonstrating how to append new HTML nodes, set new content, remove selections, and replace selections with new HTML.

```rust
use dom_query::Document;

let html_contents = r#"<!DOCTYPE html>
<html>
    <head><title>Test</title></head>
    <body>
        <div class="content">
            <p>9,8,7</p>
        </div>
        <div class="remove-it">
            Remove me
        </div>
        <div class="replace-it">
            <div>Replace me</div>
        </div>
    </body>
</html>"#;

let doc = Document::from(html_contents);

// Select the div with class "content"
let mut content_selection = doc.select("body .content");

// Append a new HTML node to the selection
content_selection.append_html(r#"<div class="inner">inner block</div>"#);
assert!(doc.select("body .content .inner").exists());

// Set a new content to the selection, replacing existing content
let mut set_selection = doc.select(".inner");
set_selection.set_html(r#"<p>1,2,3</p>"#);
assert_eq!(doc.select(".inner").html(),
    r#"<div class="inner"><p>1,2,3</p></div>"#.into());

// Remove the selection with class "remove-it"
doc.select(".remove-it").remove();
assert!(!doc.select(".remove-it").exists());

// Replace the selection with new HTML, the current selection will not change
let mut replace_selection = doc.select(".replace-it");
replace_selection.replace_with_html(r#"<div class="replaced">Replaced</div>"#);
assert_eq!(replace_selection.text().trim(), "Replace me");

// But the document will reflect the changes
assert_eq!(doc.select(".replaced").text(),"Replaced".into());


// Prepend more elements to the selection
content_selection.prepend_html(r#"<p class="third">3</p>"#);
content_selection.prepend_html(r#"<p class="first">1</p><p class="second">2</p>"#);

// Also you can insert html before selection:
let first = content_selection.select(".first");
first.before_html(r#"<p class="none">None</p>"#);
// or after:
let third = content_selection.select(".third");
third.after_html(r#"<p class="fourth">4</p>"#);

// now the added paragraphs standing in front of `div`
assert!(doc.select(r#".content > .none + .first + .second + .third + .fourth + div:has-text("1,2,3")"#).exists());

// to set a text to the selection you can use `set_html` but `set_text` is preferable:
let p_sel = content_selection.select("p");
let total_p = p_sel.length();
p_sel.set_text("test content");

assert_eq!(doc.select(r#"p:has-text("test content")"#).length(), total_p);

```

### Explanation:
- **Append HTML**:
    - The `append_html` method is used to add a new HTML node to the existing selection.
- **Set HTML**:
    - The `set_html` method replaces the existing content of the selection with new HTML.
- **Remove Selection**:
    - The `remove` method deletes the elements matching the selector from the document.
- **Replace with HTML**:
    - The `replace_with_html` method replaces the selected elements with new HTML. Note that the selection itself remains unchanged, but the document reflects the new content.
- **Prepend HTML**
    - The `prepend_html` method is used to add a new HTML node at the beginning of the existing selection.

- **Insert HTML Before/After**
    - The `before_html` method inserts HTML before each element in the selection.
    - The `after_html` method inserts HTML after each element in the selection.


## Renaming Elements Without Changing the Contents

The `dom_query` crate allows you to easily rename selected elements without changing their contents. `Selection::rename` does the same for the entire selection, while `Node::rename` does it for a single element.

```rust
use dom_query::Document;

let doc: Document = r#"<!DOCTYPE html>
<html>
<head><title>Test</title></head>
<body>
    <div class="content">
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <span>4</span>
    </div>
<body>
</html>"#.into();

let mut sel = doc.select("div.content > div, div.content > span");
// Before renaming, there are 3 `div` and 1 `span`
assert_eq!(sel.length(), 4);

sel.rename("p");

// After renaming, there are no `div` and `span` elements
assert_eq!(doc.select("div.content > div, div.content > span").length(), 0);
// But there are four `p` elements
assert_eq!(doc.select("div.content > p").length(), 4);
```


## Creating and Manipulating Elements

The `dom_query` crate allows you to create and manipulate HTML elements with ease. Below are examples demonstrating how to create new elements, set attributes, append HTML, and replace content.

```rust
use dom_query::Document;

let doc: Document = r#"<!DOCTYPE html>
<html lang="en">
<head></head>
<body>
    <div id="main">
        <p id="first">It's</p>
    <div>
</body>
</html>"#.into();

// Selecting a node we want to attach a new element
let main_sel = doc.select_single("#main");
let main_node = main_sel.nodes().first().unwrap();

// Creating a simple element
let el = doc.tree.new_element("p");
// Setting attributes
el.set_attr("id", "second");
// Setting text content
el.set_text("test");
main_node.append_child(&el);
assert!(doc.select(r#"#main #second:has-text("test")"#).exists());

// Appending a more complex element using `append_html`
main_node.append_html(r#"<p id="third">Wonderful</p>"#);
assert_eq!(doc.select("#main #third").text().as_ref(), "Wonderful");
assert!(doc.select("#first").exists());

// There is also a `prepend_child` and `prepend_html` methods which allows
// to insert content to the begging of the node.
main_node.prepend_html(r#"<p id="minus-one">-1</p><p id="zero">0</p>"#);
assert!(doc.select("#main > #minus-one + #zero + #first + #second + #third").exists());

// Replacing existing element content with new HTML using `set_html`
main_node.set_html(r#"<p id="the-only">Wonderful</p>"#);
assert_eq!(doc.select("#main #the-only").text().as_ref(), "Wonderful");
assert!(!doc.select("#first").exists());

// Completely replacing the contents of the node, 
// including itself, using `replace_with_html`
main_node.replace_with_html(
    r#"<span>Tweedledum</span> and <span>Tweedledee</span>"#
);
assert!(!doc.select("#main").exists());
assert_eq!(doc.select("span + span").text().as_ref(), "Tweedledee");

// Inserting HTML content before a certain node using `node.before_html`
let span_sel = doc.select("body > span");
let span_node = span_sel.nodes().first().unwrap();
span_node.before_html(r#"<div id="main">Main Content</div>"#);
assert!(doc.select(r#"body > #main + span:has-text("Tweedledum")"#).exists());

// Inserting HTML content after a certain node using `node.after_html`
let span_node = span_sel.nodes().last().unwrap();
span_node.after_html(r#"<div id="extra">Extra Content</div>"#);
assert!(doc.select(r#"body > span:has-text("Tweedledee") + #extra"#).exists());

// To insert nodes before or after a certain element, 
// use the `node.insert_before` and `node.insert_after` methods.
// Both methods share the same behavior as `node.append_child`.

```

#### Explanation:
- **Creating a Simple Element**:
    - Use `doc.tree.new_element()` to create a new orphan element.
    - Set attributes using `node.set_attr()`.
    - Set text content using `node.set_text()`.
    - Use `node.append_child()` to append a new child element node to the selected node.
    - Use `node.prepend_child()` to prepend a new child element node to the selected node.
    - Use `node.insert_before()` to insert a new sibling element node before the selected node.
    - Use `node.insert_after()` to insert a new sibling element node after the selected node.

- **Appending HTML**:
    - Use `append_html` to add a more complex HTML node to the existing selection.
    - This method is more convenient for adding multiple elements to the selected node.

- **Prepending HTML**:
    - Use `prepend_html` to add new HTML nodes at the beginning of the existing selection.
    - Use `prepend_child` to prepend a new or an existing element node to the selected node.

- **Setting New HTML Content**:
    - Use `set_html` to replace the existing content of the selected node with new HTML.
    - It changes the inner HTML contents of the node.
    
- **Replacing Node Contents Completely**:
    - Use `replace_with_html` to replace the entire content of the node, including the node itself.

- **Inserting HTML Before/After**:
    - Use `before_html` to insert HTML before each element in the selection.
    - Use `after_html` to insert HTML after each element in the selection.

Additionally, methods like `replace_with_html`, `set_html`, `append_html`, `prepend_html`, `before_html` and `after_html` can specify more than one element in the provided string.

### Wrapping & Unwrapping Node Elements

```rust
use dom_query::Document;

let doc: Document = r#"<!DOCTYPE html>
<html lang="en">
<head></head>
<body>
    <div id="main">
        <p id="content">It's</p>
    <div>
</body>
</html>"#.into();

let content_sel = doc.select("#content");
let content_node = content_sel.nodes().first().unwrap();

// 1. Wrapping the node with a new element
let wrapper = doc.tree.new_element("div");
wrapper.set_attr("id", "wrapper");
content_node.wrap_node(&wrapper);

assert_eq!(doc.select("#main > #wrapper > #content").length(), 1);

// 2. Wrapping the node with an HTML fragment
content_node.wrap_html(r#"<div id="sub-wrapper"><div class="adv">adv block content</div></div>"#);

// The node will be attached after `.adv` inside `#sub-wrapper`
assert_eq!(doc.select("#main > #wrapper > #sub-wrapper > .adv + #content").length(), 1);

// 3. Unwrapping the node
content_node.unwrap_node();

// Now #content is again a direct child of #wrapper
assert_eq!(doc.select("#main > #wrapper > #content").length(), 1);

// The detached #sub-wrapper still exists in the tree but is not connected
assert!(doc.select("#sub-wrapper").exists());

```


### Explanation:

- `wrap_node(&wrapper_node)` — wraps the current node inside the given node.

- `wrap_html("<html_fragment>")` — wraps the current node using the first element found in the provided HTML fragment.

- `unwrap_node()` — removes the node’s parent and moves the node up one level in the tree.

## Text Node Normalization
Node normalization is essential for merging adjacent text nodes into a single node and removing empty text nodes. This helps keep the document structure compact and organized.

```rust
use dom_query::Document;

let contents = r#"<!DOCTYPE html>
<html>
    <head><title>Test</title></head>
    <body>
        <div id="parent">
            <div id="child">Child</div>
        </div>
    </body>
</html>"#;
let doc = Document::from(contents);

// Select the node with id "child"
let child_sel = doc.select_single("#child");
let child = child_sel.nodes().first().unwrap();

// Check that the node initially has only one child
assert_eq!(child.children_it(false).count(), 1);

// Create and append new text nodes
let text_1 = doc.tree.new_text(" and a");
let text_2 = doc.tree.new_text(" ");
let text_3 = doc.tree.new_text("tail");
child.append_child(&text_1);
child.append_child(&text_2);
child.append_child(&text_3);

// Verify the text and child count before normalization
assert_eq!(child.text(), "Child and a tail".into());
assert_eq!(child.children_it(false).count(), 4);

// Normalize the node
child.normalize();

// Verify the text and child count after normalization
assert_eq!(child.children_it(false).count(), 1);
assert_eq!(child.text(), "Child and a tail".into());

```
The `normalize` method follows the [Node.normalize()](https://developer.mozilla.org/en-US/docs/Web/API/Node/normalize) specification.
This method is also available through the `Document` struct as `Document::normalize()`, which applies normalization to all text nodes within the document tree.


## Stripping Elements with `strip_elements`
`dom_query` allows you to easily remove specific elements from a node, keeping their children preserved.

```rust
use dom_query::Document;

let doc: Document = r#"<!DOCTYPE html>
<html lang="en">
<head></head>
<body>
    <div id="main">
        <p id="content">the quick brown <b>fox</b> jumps over the lazy <i>dog</i></p>
    <div>
</body>
</html>"#.into();

// 1. Select the content node
let content_sel = doc.select("#content");
let content_node = content_sel.nodes().first().unwrap();

assert_eq!(
    content_node.inner_html(),
    "the quick brown <b>fox</b> jumps over the lazy <i>dog</i>"
);

// 2. Strip specific elements (`<b>`, `<i>`) while keeping their children
content_node.strip_elements(&["b", "i"]);

// Now only text nodes remain
assert_eq!(
    content_node.inner_html(),
    "the quick brown fox jumps over the lazy dog"
);
assert_ne!(content_node.children_it(false).count(), 1);

// 3. Optional: Normalize to merge adjacent text nodes into a single node
content_node.normalize();
assert_eq!(content_node.children_it(false).count(), 1);
```

### Explanation:

- `strip_elements(["tag1", "tag2", ...])` — removes all matching elements inside the node but preserves their children.

- `normalize()` — merges adjacent text nodes into a single one, for a cleaner structure.

> **Note**: `strip_elements` only removes the specified tags themselves, preserving their content inside.