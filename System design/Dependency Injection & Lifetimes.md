
### Why use DI instead of `new`?

When you write `new MyService()` directly in a class, you create tight coupling — the caller decides _what_ it gets, _how_ it's constructed, and owns the lifetime. That means testing is painful (you can't swap in a mock), configuration is scattered, and lifetime management is manual.

With DI, you invert control: a class declares _what it needs_, and the container provides it. This gives you:

- **Swappability** — swap `SqlProductRepo` for `MockProductRepo` in tests with one line
- **Centralized config** — all wiring lives in `Program.cs`, not sprinkled through the codebase
- **Automatic lifetime management** — no manually tracking when to `Dispose()`
- **Composition** — services can receive their own dependencies without their callers caring