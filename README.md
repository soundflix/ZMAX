# ZMAX

ZMAX is a framework that bridges AX (Accessibility) APIs to Swift.

## General usage

### Getting an application element

```swift
let runningApps = NSWorkspace.shared.runningApplications
if let safariApp = runningApps.first(where: { $0.bundleIdentifier == "com.apple.Safari" }) {
	let safari = safariApp.accessibilityElement
}
```

### Getting an attribute

```swift
let windows = try safari.getAttribute(for: .windows) as [AXUIElement]
```

### Performing an action

```swift
try minimizeButton.performAction(.press)
```

### Example: Pressing minimize buttons in Safari

```swift
let runningApps = NSWorkspace.shared.runningApplications
if let safariApp = runningApps.first(where: { $0.bundleIdentifier == "com.apple.Safari" }) {
	let safari = safariApp.accessibilityElement
	
	let windows = try safari.getAttribute(for: .windows) as [AXUIElement]
	for window in windows {
		let title = try window.getAttribute(for: .title) as String
		let minimizeButton = try window.getAttribute(for: .minimizeButton) as AXUIElement
		try minimizeButton.performAction(.press)
	}
}
```

## Observing

### Example: Observing focused UI element changes in Safari

```swift
let runningApps = NSWorkspace.shared.runningApplications
if let safariApp = runningApps.first(where: { $0.bundleIdentifier == "com.apple.Safari" }) {
	let safari = safariApp.accessibilityElement
	
	let observer = try safariApp.newAccessibilityObserver()
	try observer.observe(element: safari, notification: .focusedUiElementChanged) { element, changes in
		if let role = try? element.getAttribute(for: .role) as String {
			print("focused element changed to \(role).")
		}
	}
}

self.safariObserver = observer //keep it!
```
## Using original APIs

Since this framework just extends original AXUIElement type, you can use original APIs without casting.

```swift
let axError = AXUIElementPerformAction(minimizeButton, (kAXPressAction as CFString))
```
## Error

This framework extends AXError with LocalizedError protocol.

### Throwing

You can throw an AXError as a Swift error.

```swift
let axError = AXUIElementPerformAction(minimizeButton, (kAXPressAction as CFString))
	if (axError != .success) {
		throw axError
	}
}
```
### Throwing with method

By using `AXError.throwIfNotSuccess()` method, you can call existing functions with `try` keyword.

```swift
try AXUIElementPerformAction(finder.accessibilityElement, kAXPressAction as CFString).throwIfNotSuccess()
```

### Localizing messages

This framework doesn't contain any localization for error messages. But you can provide and customize messages.

```swift
AXError.customLocalizedDescriptionHandlers.append { error in
	NSLocalizedString(String(format: "AXError_%i", error.rawValue), comment: "")
}
```
