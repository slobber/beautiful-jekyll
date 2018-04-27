---
layout: post
title: 纯干货 Swift
image: /assets/swift/puredry.logo.jpg
category: [ swift ]
---

## String

### substring

```swift
extension String {
    func substring(with nsrange: NSRange) -> String? {
        guard let range = Range(nsrange, in: self) else { return nil }
        return self[range] == nil ? nil : String(self[range])
    }

    func substring(from: Int, to: Int)
}
```