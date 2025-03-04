# WASM32 Compilation
When compiling dom_query to WebAssembly (target `wasm32-unknown-unknown`) using `wasm-pack`, you may encounter runtime panics related to memory allocation, such as:

```
panicked at 'assertion failed: psize <= size + max_overhead'
```

This issue currently occurs due to compatibility problems between the latest versions of the `selectors` crate and the `dlmalloc` crate. The issue specifically manifests when using pseudo-elements, including `selectors`' own pseudo-elements like `:not` and `:has`.

If you **must** compile dom_query for a wasm32 application, consider using an alternative to `dlmalloc`. The following allocators have been tested and work successfully:
- wee_alloc
- lol_alloc
- mini-alloc (fast, but it never deallocates memory)
- alloc_cat

### Solution: 

1. Add mini-alloc to your Cargo.toml:
```toml
[dependencies]
alloc_cat = "1.0.0"
```

2. Set your favorite allocator as the global allocator in your lib.rs or main.rs:

```rust
#[cfg(target_arch = "wasm32")]
#[global_allocator]
pub static ALLOC: &alloc_cat::AllocCat = &alloc_cat::ALLOCATOR;
```

3. Build or test your WebAssembly project
