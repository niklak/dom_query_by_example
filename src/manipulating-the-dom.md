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
```

#### Explanation:
- Creating a Simple Element:
    - Use `doc.tree.new_element()` to create a new element.
    - Set attributes using `node.set_attr()`.
    - Set text content using `node.set_text()`.
    - Append the new element to the selected node using `node.append_child()`.
- Appending HTML:
    - Use `append_html` to add a more complex HTML node to the existing selection.
    - This method is more convenient for adding multiple elements to the selected node.

- Setting New HTML Content:
    - Use `set_html` to replace the existing content of the selected node with new HTML.
    - It changes the inner HTML contents of the node.
- Replacing Node Contents Completely:
    - Use `replace_with_html` to replace the entire content of the node, including the node itself.

Additionally, methods like `replace_with_html`, `set_html`, and `append_html` can specify more than one element in the provided string.
