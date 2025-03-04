# Accessing and Manipulating the element's attributes

The `dom_query` crate provides several methods for accessing and manipulating the attributes of an HTML element.

> [!NOTE]
> All methods listed below apply to both Selection and Node.

### Getting an attribute value

You can use the `attr()` method to retrieve the value of a specific attribute. If the attribute does not exist, it will return `None`.
You can use the `attr_or()` method to retrieve the value of a specific attribute, and return a default value if the attribute does not exist.


```rust
use dom_query::Document;

let html = r#"<!DOCTYPE html>
<html>
    <head><title>Test</title></head>
    <body><input hidden="" id="k" class="important" type="hidden" name="k" data-k="100"></body>
</html>"#;

let doc = Document::from(html);

let mut input_selection = doc.select("input[name=k]");

let val = input_selection.attr("data-k").unwrap();
assert_eq!(val.to_string(), "100");

// try to get an attribute that does not exist
let val_or = input_selection.attr_or("data-l", "0");
assert_eq!(val_or.to_string(), "0");
```

### Getting the class attribute

You can use the `class()` method to retrieve the value of the `class` attribute.
If the `class` attribute does not exist, the method returns `None`.

For `Selection`, the class method will return the value of the `class` attribute of the first element in the selection.
For `Node`, the class method will return the value of the `class` attribute of the current element.

```rust
use tendril::StrTendril;

let input_selection = doc.select("input");
let class: Option<StrTendril> = input_selection.class();
assert_eq!(class, Some("important".into()));
```

### Getting the id attribute

Everything works the same way as with the `class` attribute,
but the method for `Selection` is called `id()`, while for `Node`, it is called `id_attr()`.

```rust
use tendril::StrTendril;

let input_selection = doc.select("input");
let id_attr: Option<StrTendril> = input_selection.id();
assert_eq!(id_attr, Some("k".into()));

let input_node = input_selection.first().unwrap();
let id_attr: Option<StrTendril> = input_node.id_attr();
assert_eq!(id_attr, Some("k".into()));

```

### Removing an attribute

You can use the `remove_attr()` method to remove a specific attribute from the element.
If it called from the `Selection` then it will remove an attribute from all elements in the selection.

```rust
input_selection.remove_attr("data-k");
```

### Removing multiple attributes

You can use the `remove_attrs()` method to remove multiple attributes from the element.
If it called from the `Selection` then it will remove all listed attributes from all elements in the selection.

```rust
input_selection.remove_attrs(&["id", "class"]);
```

### Setting an attribute value

You can use the `set_attr()` method to set the value of a specific attribute.
If it called from the `Selection` then it will set an attribute to all elements in the selection.

```rust
input_selection.set_attr("data-k", "200");
```

### Checking if an attribute exists
You can use the `has_attr()` method to check if a specific attribute exists on the element.
If it called from the `Selection` then it will check if an attribute exists on the first element in the selection.

```rust
let is_hidden = input_selection.has_attr("hidden");
assert!(is_hidden);
```

### Removing all attributes
You can use the `remove_all_attrs()` method to remove all attributes from the element.
If it called from the `Selection` then it will remove all attributes from all elements in the selection.

```rust
input_selection.remove_all_attrs();
assert_eq!(input_selection.html(), r#"<input>"#.into());
```
