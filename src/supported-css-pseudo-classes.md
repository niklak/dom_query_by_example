### **Supported CSS pseudo-classes in `dom_query`**

**Implementation with `selectors`:**

* `:empty`
* `:first-child`
* `:last-child`
* `:has`
* `:is`
* `:where`
* `:last-of-type`
* `:not`
* `:only-child`
* `:only-of-type`
* `:nth-child`
* `:nth-last-child`

**Implementation with `dom_query`:**
* `:any-link`
* `:link`
* `:has-text`
* `:contains`
* `:only-text`

### Notes
`:has-text` – checks whether one of children nodes has specific text.

`:contains` – checks whether the combined text of all child nodes contains specific text.

`:only-text` - checks whether the element contains only a single text node, with no other child nodes.
