---
title: "Instagram DM Wrapper"
tagline: "A focused, distraction-free Instagram Direct Messages (DM) wrapper built with Flutter."
website: ""
skills: ["Flutter", "Dart", "flutter_inappwebview", "CSS Injection", "URL Interception"]
---

A dedicated "DM-only" client for Instagram designed to eliminate social media distractions. This app wraps the Instagram web interface and uses aggressive filtering to ensure you only see your messages.

## The Problem
Standard social media apps are designed to keep you scrolling. Even when you just want to check a message, you're often pulled into the Feed, Reels, or Explore page, leading to hours of lost productivity.

## The Solution
The Instagram DM Wrapper provides a focused environment by:
- **Direct Boot**: Automatically navigates to the DM inbox on startup.
- **Aggressive Distraction Blocking**: Removes the Feed, Reels tab, and Explore tab using custom CSS.
- **Navigation Control**: Intercepts URL requests to block accidental navigation to distracting parts of the site.

## Key Features

### DM Focused
The app acts as a standalone messenger. By targeting `instagram.com/direct/inbox/` directly, it bypasses the main landing page.

### CSS Injection
I implemented a robust CSS injection system that hides UI elements responsible for distractions:
- Hides the home navigation link.
- Removes the Reels and Explore sidebar/bottom bar items.
- Cleans up the interface for a minimal, messenger-like look.

### Navigation Interception
Using `flutter_inappwebview`, the app monitors every URL request. If a user clicks a link (or a script tries to redirect) to `/reels/` or `/explore/`, the app blocks the request and keeps the user within the DM context.

### Immersive & Native Feel
- **Edge-to-Edge Design**: Full-screen experience without browser chrome.
- **Android Back Gesture**: Correctly handles the system back button to navigate internal web history instead of closing the app.
- **Dark Mode**: Optimized for a sleek, dark aesthetic.

## Technical Implementation

### WebView Configuration
The core of the app uses `InAppWebView` with specialized settings:
- `incognito: true` (optional) for privacy.
- `transparentBackground: true` for seamless UI integration.

### Back Button Handling
```dart
onWillPop: () async {
  if (await webViewController.canGoBack()) {
    webViewController.goBack();
    return false;
  }
  return true;
},
```

## Impact
This project has significantly reduced my own screen time by allowing me to stay connected with friends without falling into the "infinite scroll" trap. It's a prime example of using technology to regain control over digital habits.

## Technologies Used
Flutter, Dart, flutter_inappwebview, CSS, JavaScript (for DOM manipulation).
