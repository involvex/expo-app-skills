---
name: apple-targets
description: Create Apple extension targets like widgets, app clips, share extensions, and more using @bacons/apple-targets
---

## Important: Custom Builds Required

**Apple targets require custom native builds.** Your app will no longer work in Expo Go.

Only use this skill when you specifically need iOS extensions like widgets, app clips, or share extensions. If you're just building a standard app, use Expo Go with `npx expo start` instead.

## What are Apple Targets?

Apple targets are additional native extensions that run alongside your main app, including home screen widgets, live activities, app clips, share extensions, Safari extensions, watch apps, and more. The `@bacons/apple-targets` package provides an Expo Config Plugin to generate and manage these targets while maintaining Continuous Native Generation.

## Requirements

- CocoaPods 1.16.2+ (Ruby 3.2.0+)
- Xcode 16+
- macOS 15 Sequoia+
- Expo SDK 53+

## Creating a Target

Run `npx create-target` in your Expo project root:

```bash
# Interactive mode - select target type
npx create-target

# Direct creation
npx create-target widget
npx create-target clip
npx create-target share
```

This will:
1. Generate target files in the `/targets` directory
2. Install `@bacons/apple-targets`
3. Add the Config Plugin to your `app.json`

## Supported Target Types

| Type | Command | Purpose |
|------|---------|---------|
| widget | `npx create-target widget` | Home screen widgets and live activities |
| clip | `npx create-target clip` | App Clip experiences |
| action | `npx create-target action` | Share sheet actions |
| share | `npx create-target share` | Share extensions |
| safari | `npx create-target safari` | Safari extensions |
| watch | `npx create-target watch` | Apple Watch companion apps |
| intent | `npx create-target intent` | Siri intent extensions |
| intent-ui | `npx create-target intent-ui` | Siri intent UI extensions |
| notification-content | `npx create-target notification-content` | Rich notification content |
| notification-service | `npx create-target notification-service` | Notification processing |
| spotlight | `npx create-target spotlight` | Spotlight index extensions |
| bg-download | `npx create-target bg-download` | Background download extensions |
| credentials-provider | `npx create-target credentials-provider` | Credentials provider extensions |
| account-auth | `npx create-target account-auth` | Account authentication extensions |

## Folder Structure

After creating a target:

```
targets/
  my-widget/
    expo-target.config.js    # Target configuration
    index.swift              # Main SwiftUI entry point
    assets/                  # Linked resources
  _shared/                   # Files linked to ALL targets
```

## Target Configuration

Each target has an `expo-target.config.js` (or `.json`) file:

```js
// targets/my-widget/expo-target.config.js
module.exports = {
  type: "widget",
  name: "MyWidget",

  // Optional: custom bundle identifier
  // Defaults to: <app-bundle-id>.<target-folder-name>
  bundleIdentifier: "com.example.app.widget",

  // iOS deployment target (defaults to 18.0)
  deploymentTarget: "17.0",

  // Special colors for build settings
  colors: {
    $accent: "#FF6B6B",           // Widget tint color
    $widgetBackground: "#FFFFFF", // Widget background
  },

  // Frameworks to link
  frameworks: ["SwiftUI", "WidgetKit"],

  // Custom entitlements
  entitlements: {
    "com.apple.security.application-groups": ["group.com.example.app"]
  }
};
```

### Dynamic Configuration

Use a function to access Expo config values:

```js
// targets/my-widget/expo-target.config.js
module.exports = (config) => ({
  type: "widget",
  name: "MyWidget",
  entitlements: {
    // Mirror app's App Groups
    "com.apple.security.application-groups":
      config.ios?.entitlements?.["com.apple.security.application-groups"] ?? []
  }
});
```

## App Groups (Data Sharing)

Widgets and extensions run in separate processes. Use App Groups to share data:

### 1. Configure App Groups in app.json

```json
{
  "expo": {
    "ios": {
      "appleTeamId": "YOUR_TEAM_ID",
      "entitlements": {
        "com.apple.security.application-groups": ["group.com.example.app"]
      }
    }
  }
}
```

### 2. Mirror in Target Config

```js
// targets/my-widget/expo-target.config.js
module.exports = (config) => ({
  type: "widget",
  entitlements: {
    "com.apple.security.application-groups":
      config.ios?.entitlements?.["com.apple.security.application-groups"] ?? []
  }
});
```

### 3. Use ExtensionStorage in React Native

```tsx
import { ExtensionStorage } from "@bacons/apple-targets";

// Create storage instance with your App Group
const storage = new ExtensionStorage("group.com.example.app");

// Write data
storage.set("userName", "John");
storage.set("score", 42);
storage.set("settings", { theme: "dark", notifications: true });

// Reload widgets to reflect changes
ExtensionStorage.reloadWidget();

// For iOS 18+ Control Widgets
ExtensionStorage.reloadControls();
```

### 4. Read in SwiftUI Widget

```swift
import WidgetKit
import SwiftUI

struct MyProvider: TimelineProvider {
    func getTimeline(in context: Context, completion: @escaping (Timeline<MyEntry>) -> Void) {
        let defaults = UserDefaults(suiteName: "group.com.example.app")
        let userName = defaults?.string(forKey: "userName") ?? "Guest"
        let score = defaults?.integer(forKey: "score") ?? 0

        let entry = MyEntry(date: Date(), userName: userName, score: score)
        let timeline = Timeline(entries: [entry], policy: .atEnd)
        completion(timeline)
    }

    func placeholder(in context: Context) -> MyEntry {
        MyEntry(date: Date(), userName: "User", score: 0)
    }

    func getSnapshot(in context: Context, completion: @escaping (MyEntry) -> Void) {
        completion(MyEntry(date: Date(), userName: "User", score: 0))
    }
}

struct MyEntry: TimelineEntry {
    let date: Date
    let userName: String
    let score: Int
}
```

## Building and Running

```bash
# 1. Set Apple Team ID in app.json
# 2. Generate native project
npx expo prebuild -p ios --clean

# 3. Open in Xcode
xed ios

# 4. Develop in expo:targets folder (changes persist outside ios/)

# 5. Build and run
npx expo run:ios
```

## Widget Example

### SwiftUI Widget (targets/my-widget/index.swift)

```swift
import WidgetKit
import SwiftUI

struct MyWidgetEntryView: View {
    var entry: MyProvider.Entry

    var body: some View {
        VStack {
            Text("Hello, \(entry.userName)!")
                .font(.headline)
            Text("Score: \(entry.score)")
                .font(.subheadline)
        }
        .containerBackground(.fill.tertiary, for: .widget)
    }
}

@main
struct MyWidget: Widget {
    let kind: String = "MyWidget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: MyProvider()) { entry in
            MyWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("Shows your current score.")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```

## Live Activities

For live activities, add the necessary attributes:

```swift
import ActivityKit
import WidgetKit
import SwiftUI

struct MyActivityAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        var progress: Double
        var status: String
    }

    var name: String
}

struct MyLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: MyActivityAttributes.self) { context in
            // Lock screen presentation
            VStack {
                Text(context.attributes.name)
                ProgressView(value: context.state.progress)
                Text(context.state.status)
            }
            .padding()
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading) {
                    Text(context.attributes.name)
                }
                DynamicIslandExpandedRegion(.trailing) {
                    Text("\(Int(context.state.progress * 100))%")
                }
            } compactLeading: {
                Text(context.attributes.name)
            } compactTrailing: {
                Text("\(Int(context.state.progress * 100))%")
            } minimal: {
                Text("\(Int(context.state.progress * 100))%")
            }
        }
    }
}
```

## App Clips

App Clips require additional setup:

### 1. Create the target

```bash
npx create-target clip
```

### 2. Configure for JS bundling

```js
// targets/my-clip/expo-target.config.js
module.exports = {
  type: "clip",
  name: "MyClip",
  exportJs: true,  // Bundle React Native code
};
```

### 3. Set up Associated Domains

```json
{
  "expo": {
    "ios": {
      "associatedDomains": ["appclips:example.com"]
    }
  }
}
```

### 4. Configure AASA file on your server

```json
{
  "appclips": {
    "apps": ["TEAMID.com.example.app.Clip"]
  }
}
```

## Share Extensions

```bash
npx create-target share
```

```js
// targets/share/expo-target.config.js
module.exports = {
  type: "share",
  name: "ShareExtension",
  exportJs: true,  // For React Native share extensions
};
```

## Safari Extensions

```bash
npx create-target safari
```

Place extension resources in `targets/safari/assets/`.

## Dark Mode Colors

Define responsive colors in your target config:

```js
module.exports = {
  type: "widget",
  colors: {
    $accent: { light: "#007AFF", dark: "#0A84FF" },
    $widgetBackground: { light: "#FFFFFF", dark: "#1C1C1E" },
    customColor: { light: "#E4975D", dark: "#3E72A0" },
  }
};
```

Use in SwiftUI:

```swift
Color("customColor")  // Automatically adapts to light/dark mode
```

## EAS Build Configuration

For EAS Build, configure extensions in `app.json`:

```json
{
  "expo": {
    "extra": {
      "eas": {
        "build": {
          "experimental": {
            "ios": {
              "appExtensions": [
                {
                  "bundleIdentifier": "com.example.app.widget",
                  "targetName": "my-widget",
                  "entitlements": {
                    "com.apple.security.application-groups": ["group.com.example.app"]
                  }
                }
              ]
            }
          }
        }
      }
    }
  }
}
```

## Custom CocoaPods Configuration

Add a `pods.rb` file at the repo root for target-specific pod settings:

```ruby
# pods.rb
target 'my-widget' do
  pod 'SomeWidgetDependency'
end
```

## Development Workflow

1. Modify `expo-target.config.js` or `app.json`
2. Run `npx expo prebuild -p ios --clean`
3. Open Xcode: `xed ios`
4. Edit SwiftUI code in `expo:targets/<target-name>`
5. Build and run from Xcode to test the target

Changes in the virtual `expo:targets` folder persist outside the generated `ios` directory, so they survive `prebuild --clean`.

## Faster Iteration

Use the blank template for quicker rebuilds:

```bash
npx expo prebuild -p ios --clean --template ./node_modules/@bacons/apple-targets/prebuild-blank.tgz
```

## Tips

- Start with a widget to learn the workflow before trying complex targets
- App Clips are powerful but complex - ensure AASA file and associated domains are correct
- Use `ExtensionStorage.reloadWidget()` after updating shared data
- Test widgets on device - simulator has limited widget support
- Keep widget code simple - they have memory and CPU constraints
- Use the `_shared` folder for code/assets needed by multiple targets
