---
layout: post
title: Building a Distraction-Free Instagram DM Client with Flutter
---

Social media platforms are masterpieces of attention engineering. Sometimes, you just want to reply to a friend's message without being bombarded by the Feed, Reels, or Explore page. Last week, I decided to take control of my digital environment by building a focused Instagram DM wrapper using Flutter.

## The Goal
The objective was simple: create an app that provides access to Instagram Direct Messages but makes it impossible to access anything else.

## The Technical Strategy

To achieve this, I used `flutter_inappwebview` and focused on three technical pillars: URL Interception, CSS Injection, and Navigation Control.

### 1. Directing the Flow
The first step was ensuring the app always starts at the inbox.
```dart
initialUrlRequest: URLRequest(
  url: WebUri("https://www.instagram.com/direct/inbox/"),
),
```

### 2. Digital Surgery via CSS Injection
This is where the magic happens. By injecting CSS once the page loads, we can "ghost" distracting elements. 
I targeted the selectors for the home icon, search/explore tab, and reels buttons:

```css
/* Example of the injected CSS */
a[href="/"], a[href="/explore/"], a[href="/reels/"] {
  display: none !important;
}
div[role="dialog"] ... { /* Hiding popups that lead away */ }
```

In Flutter, this is executed in the `onLoadStop` handler:
```dart
onLoadStop: (controller, url) async {
  await controller.injectCSSCode(source: """
    /* My custom CSS rules */
  """);
}
```

### 3. Hard Blocking with URL Interception
CSS hiding is great, but what if a link is clicked? I implemented a `shouldOverrideUrlLoading` handler to act as a firewall. 

```dart
shouldOverrideUrlLoading: (controller, navigationAction) async {
  var uri = navigationAction.request.url!;
  if (uri.path.contains("/reels/") || uri.path.contains("/explore/")) {
    return NavigationActionPolicy.CANCEL; // Block the distraction
  }
  return NavigationActionPolicy.ALLOW;
},
```

## Handling the "Native" Experience
To make it feel like a real app rather than a website in a box, I had to ensure the Android back button worked as expected. Without manual handling, the back gesture would often exit the app. By checking `canGoBack()`, the app now moves through message threads correctly.

## The Result
The result is a clean, dark-themed messenger. It boots instantly into my conversations. There is no feed to scroll, no videos to watch, and no search bar to explore. It has turned a major source of distraction into a functional tool.

## Key Learnings
- **Web scraping vs. Web Wrapping**: Wrapping with aggressive UI manipulation is often more resilient than building a custom client from scratch using unofficial APIs.
- **CSS Specificity**: When working with injected styles, `!important` is your best friend to override the complex stylesheets of modern web apps.
- **Intentional Design**: Building tools for ourselves is one of the best ways to learn how to solve real-world problems.

This project was a fun weekend challenge that has already had a positive impact on my productivity. It goes to show that with a little bit of code, we can reshape the digital tools we use every day.
