% `Drop`

The `Drop` [trait][trait] is a special one, supported by the Rust compiler. It allows
you to define a ‘destructor’, some code that runs when your value is about to go out
of scope.

[trait]: traits.html

For example, here’s a type which prints out a message when it is dropped:

```rust
struct Noisy;

impl Drop for Noisy {
    fn drop(&mut self) {
        println!("Goodbye, cruel world!");
    }
}

{
    let n = Noisy;

    // stuff happens

} // `Drop::drop` gets called here
```

When `n` goes out of scope, “Goodbye, crule world!” is printed.

This feature is very useful for ensuring safety. For example, let’s consider
the [DynamicLibrary][dl] library. It has a [`struct`] which represents a
handle to the library being loaded:

[dl]: ../std/dynamic_lib/index.html
[struct]: structs.html

```rust
struct DynamicLibrary {
    handle: *mut u8
}
```

That `*mut u8` is a [raw pointer][rawpointers]. We’ll use it to manage the
library handle. But, as a raw pointer, it won’t help us with ownership, and so
we need to clean up the `handle` ourselves. We can do that with `Drop`:

[rawpointers]: raw-pointers.html

```rust,ignore
impl Drop for DynamicLibrary {
    fn drop(&mut self) {
        // real code also checks for errors
        unsafe {
            dl::close(self.handle)
        }
    }
}
```

That way, whenever a library is loaded, we make sure that the handle is closed
whenever we stop using the library.
