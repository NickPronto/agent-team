# Swift Standards (Native iOS Modules)

## Scope
This applies to native iOS modules/extensions only. The primary app is React Native — use Swift only when bridging to native APIs not available in RN.

## Key Rules
- Swift 5.9+, async/await over completion handlers
- `struct` over `class` unless you need reference semantics
- Explicit access control: mark everything `private` by default, promote only when needed
- No force unwraps (`!`) in production code — use `guard let` or `if let`
- `Codable` for all JSON serialization — no manual `JSONSerialization`

## React Native Bridge
- Use `@objc` annotations for methods exposed to RN
- Module name must match `RCT_EXPORT_MODULE()` declaration
- Callbacks must be called exactly once — use `resolve`/`reject` pattern
- Heavy work goes on a background queue: `DispatchQueue.global(qos: .userInitiated).async`

## Error Handling
- Use `throws` + `do/catch` — never `fatalError` in bridged code
- Translate Swift errors to RN-friendly strings before passing to `reject`
