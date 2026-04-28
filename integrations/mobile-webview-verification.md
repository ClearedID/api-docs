# Mobile WebView: file upload, camera, and identity flows

> **Cleared API documentation — client integrations**  
> This page lives in the **[api-docs](../README.md)** repository. It does **not** document REST endpoints. It explains how **your mobile app** (typically **Flutter** + **`webview_flutter`**) must configure an embedded **WebView** so customers can complete **Cleared-hosted** identity and address verification inside your app.

**Audience:** Engineers embedding **Cleared-hosted** identity or onboarding pages in a **Flutter** app using **`webview_flutter`** (the same stack Cleared ships for onboarding and in-app verification). Other native stacks are noted only where the **Android WebView contract** differs by language, not as a second full tutorial.

**Purpose:** This is an **implementation guide**. It describes the **web APIs** those pages typically use (`<input type="file">`, `capture`, MIME filters, optional `getUserMedia`), the **three permission layers** (manifest / Info.plist, OS runtime, WebView callbacks — [§5](#5-permissions-what-must-be-entitled-for-the-webview)), and the **Flutter** hooks (`webview_flutter_android`). **Copy-paste Dart** is in [§7](#7-flutter-implementation-guide); the **QA checklist** is [§12](#12-qa-checklist).

---

## Table of contents

1. [Goals and scope](#1-goals-and-scope)
2. [Background: Android WebView and file uploads](#2-background-android-webview-and-file-uploads)
3. [Android: required native behaviour](#3-android-required-native-behaviour)
4. [The file URI contract (critical)](#4-the-file-uri-contract-critical)
5. [Permissions (entitling the WebView)](#5-permissions-what-must-be-entitled-for-the-webview)
6. [Android manifest and package visibility](#6-android-manifest-and-package-visibility)
7. [Flutter implementation guide](#7-flutter-implementation-guide)
8. [Flutter vs other stacks (why Kotlin is not the focus here)](#8-flutter-vs-other-stacks-why-kotlin-is-not-the-focus-here)
9. [iOS notes](#9-ios-notes)
10. [Hosted page behaviour (summary)](#10-hosted-page-behaviour-summary)
11. [Troubleshooting](#11-troubleshooting)
12. [QA checklist](#12-qa-checklist)
13. [Security and privacy notes](#13-security-and-privacy-notes)

---

## 1. Goals and scope

### Goals

- When the user taps **Take photo**, **Choose file**, or equivalent in the **hosted web page**, your app should:
  - Open the **system camera** or **document / gallery picker** as appropriate.
  - Return the chosen **image or video** to the WebView’s file input so JavaScript receives a `change` event with a usable `File` and the flow can continue.
- The hosted experience may use:
  - **`<input type="file">`** with `accept` and `capture` (common for mobile-friendly capture).
  - **`navigator.mediaDevices.getUserMedia`** for in-page camera preview (separate from file upload; see [§3.2](#32-getusermedia-vs-file-input)).

### Out of scope

- Backend session design, tokens, and release coordination for the hosted site (work with Cleared on those separately).
- Desktop browsers (file APIs work without a native bridge).

---

## 2. Background: Android WebView and file uploads

### 2.1 No file chooser bridge (Android)

On **Android**, Chromium WebView **does not** automatically connect `<input type="file">` to the system picker. You must implement **`WebChromeClient.onShowFileChooser`** and return one or more **`Uri`** values the engine can pass to the page.

**Symptom:** User taps upload or take photo; **nothing happens** or the control does not respond.

**Fix:** Register a file-chooser handler on the WebView that loads your hosted URL.

### 2.2 Invalid URI: raw filesystem path

If the handler returns a **plain absolute path** (e.g. `/data/user/0/.../cache/photo.jpg`) instead of a **`file:`** or **`content:`** URI, Chromium may log:

```text
The file choice request has an invalid Uri: /data/user/0/.../cache/....jpg
```

The page **does not** receive a valid file; the step **does not** complete.

**Fix:** Return **`Uri.fromFile(file)`** / **`file://...`** or a **`content://`** URI from **`FileProvider`**. Do not return an unqualified path string.

### 2.3 Activity lifecycle

Opening the camera often moves your activity through **`onStop`**. Ensure:

- Results are delivered to the **same** callback your WebView integration expects.
- Optional: handle **configuration changes** if the user rotates the device during capture.

**Flutter:** return **`Future<List<String>>`** from the file-selector callback; empty list = cancel.

### 2.4 Permissions

Without **camera** or **media / storage** permissions where required, pickers may fail or appear to do nothing. Request permissions **before** launching camera or gallery when the OS requires it.

---

## 3. Android: required native behaviour

### 3.1 File chooser bridge

When the WebView requests a file:

1. Read **MIME types** (`accept`), **capture** flag, and **single vs multiple** selection.
2. Launch the right UI:
   - **Camera** when `capture` is set and the request is image-only (often **`environment`** for ID scans, **`user`** for selfie—match the page).
   - **Gallery / document picker** when there is no capture flag or the page allows general file pick.
   - **Video** when `accept` includes `video/*` (duration rules, if any, are enforced in the page).
3. On success, return **`List<Uri>`** (or string **`file:`** / **`content:`** URIs, depending on your wrapper).
4. On cancel, return **`null`** or an **empty list** per your API (Flutter **`webview_flutter_android`:** empty list = cancel).

### 3.2 `getUserMedia` vs file input

| Mechanism | Native hook |
|-----------|----------------|
| `<input type="file">` | `onShowFileChooser` |
| `getUserMedia` / in-page preview | `onPermissionRequest` (plus manifest permissions) |

Hosted verification pages may use **both**. Delegate **permission requests** to the OS, then allow the WebView to grant camera (and microphone if needed) according to your policy.

---

## 4. The file URI contract (critical)

Paths from **`ImagePicker`**, **`FilePicker`**, camera **`Intent`** extras, etc. must be turned into **URIs** before handing them to WebView:

- If the value already starts with **`content:`** or **`file:`**, use it as-is.
- Otherwise treat it as a filesystem path; in **Dart** use **`Uri.file(path).toString()`**. On raw Android, equivalents are **`Uri.fromFile`** / **`FileProvider`** URIs.

Android WebView validates the value as a **Uri**; a path with **no scheme** is invalid.

---

## 5. Permissions: what must be “entitled” for the WebView

The WebView does **not** inherit camera, microphone, location, or storage by itself. You need **three aligned layers**; missing any layer produces “nothing happens” or a frozen preview with **no** useful JS error.

### 5.1 Layer 1 — OS declaration (Android manifest / iOS Info.plist)

**Android (`AndroidManifest.xml`):** declare every capability the hosted page may use, for example **`CAMERA`**, **`RECORD_AUDIO`**, **`READ_MEDIA_IMAGES`**, **`READ_MEDIA_VIDEO`**, **`READ_EXTERNAL_STORAGE`** (with `maxSdkVersion` where appropriate), and **`ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION`** if the page calls **`navigator.geolocation`**. Without the matching **`<uses-permission>`** entry, runtime requests can fail immediately.

**iOS (`Info.plist`):** add the **usage description** keys (`NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, `NSPhotoLibraryUsageDescription`, **`NSLocationWhenInUseUsageDescription`** when location is used, etc.). These are required for the system sheet to appear; they are separate from optional **Runner.entitlements** (e.g. App Groups) unless your app adds other capabilities.

### 5.2 Layer 2 — Runtime OS grant

After declarations, the user must still **grant at runtime** (Flutter: **`permission_handler`** or your own flow). Request **before** you open the camera app or before the page expects **`getUserMedia`** to succeed.

### 5.3 Layer 3 — WebView / Chromium second gate (Flutter)

Even with OS permission granted, Chromium may **block the page** until the **host app** answers WebView callbacks:

| Web need | Flutter (`webview_flutter_android`) |
|----------|-------------------------------------|
| In-page camera / mic (`getUserMedia`) | **`WebViewController.fromPlatformCreationParams(..., onPermissionRequest: …)`** — request OS permission if needed, then **`await request.grant()`** (or deny). Skipping **`grant()`** leaves the page without a stream. |
| Page geolocation | **`AndroidWebViewController.setGeolocationPermissionsPromptCallbacks`** — request **`Permission.locationWhenInUse`**, then return **`GeolocationPermissionsResponse(allow: true, …)`**. Without this, **`navigator.geolocation`** fails in WebView even if manifest + OS location are OK. |
| File / camera picker | **`setOnShowFileSelector`** (see [§3](#3-android-required-native-behaviour), [§7](#7-flutter-implementation-guide)). |

Treat **Layer 3** as **entitling the WebView process** to use the capability for that **origin**, not only entitling the Flutter app in the abstract.

### 5.4 Quick reference — Android runtime `permission_handler` targets

| OS permission | Typical use |
|----------------|-------------|
| `Permission.camera` | Camera capture, `getUserMedia` video |
| `Permission.microphone` | In-page recording with audio |
| `Permission.photos` / storage reads | Gallery / file pickers (platform-specific APIs) |
| `Permission.locationWhenInUse` | Geolocation in WebView (with §5.3 callback) |

See [§6](#6-android-manifest-and-package-visibility) for manifest examples and [§7.6](#76-integration-checklist) for the Flutter wiring checklist.

---

## 6. Android manifest and package visibility

### 6.1 Manifest permissions

Example baseline (adjust to your security and product requirements):

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="false" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<!-- Gallery / file input (API 33+) -->
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
<!-- If the hosted page uses navigator.geolocation (e.g. address verification): -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### 6.2 Package visibility (`<queries>`)

From **Android 11** onward, package visibility can block intents to other apps. If camera or gallery **never opens** and logcat shows **resolveActivity** failures, add the appropriate **`<queries>`** entries per [Package visibility](https://developer.android.com/training/package-visibility). For **`http`/`https`** helpers (e.g. “open in browser”), you may also need **`VIEW`** intents for those schemes.

---

## 7. Flutter implementation guide

The sections below are a **self-contained worked example**: controller creation, URI normalisation, file selection, and safe **`loadRequest`** timing. Adapt names and structure to your app.

### 7.1 Dependencies (`pubspec.yaml`)

```yaml
dependencies:
  webview_flutter: ^4.13.1
  webview_flutter_android: ^4.10.1
  webview_flutter_wkwebview: ^3.23.0
  file_picker: ^10.3.2
  image_picker: ^1.0.4
  permission_handler: ^12.0.1
```

Imports for the snippets:

```dart
import 'dart:io' show Platform;

import 'package:file_picker/file_picker.dart';
import 'package:flutter/foundation.dart';
import 'package:image_picker/image_picker.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:webview_flutter/webview_flutter.dart';
import 'package:webview_flutter_android/webview_flutter_android.dart';
import 'package:webview_flutter_wkwebview/webview_flutter_wkwebview.dart';
```

### 7.2 Create the `WebViewController` (permissions + Android file selector)

Use **`WebViewController.fromPlatformCreationParams`** for **`onPermissionRequest`**. On Android, cast **`controller.platform`** to **`AndroidWebViewController`** and call **`setOnShowFileSelector`**.

```dart
Future<WebViewController> createHostedWebViewController() async {
  const defaultParams = PlatformWebViewControllerCreationParams();
  late final PlatformWebViewControllerCreationParams params;

  if (!kIsWeb && WebViewPlatform.instance is WebKitWebViewPlatform) {
    params = WebKitWebViewControllerCreationParams
        .fromPlatformWebViewControllerCreationParams(
      defaultParams,
      allowsInlineMediaPlayback: true,
      mediaTypesRequiringUserAction: const <PlaybackMediaTypes>{},
    );
  } else {
    params = defaultParams;
  }

  final controller = WebViewController.fromPlatformCreationParams(
    params,
    onPermissionRequest: (WebViewPermissionRequest request) async {
      for (final t in request.types) {
        if (t == WebViewPermissionResourceType.camera) {
          await Permission.camera.request();
        } else if (t == WebViewPermissionResourceType.microphone) {
          await Permission.microphone.request();
        }
      }
      await request.grant();
    },
  );

  await controller.setJavaScriptMode(JavaScriptMode.unrestricted);

  if (!kIsWeb && Platform.isAndroid) {
    final platform = controller.platform;
    if (platform is AndroidWebViewController) {
      await platform.setOnShowFileSelector(_pickFilesForAndroidWebView);
      await platform.setGeolocationPermissionsPromptCallbacks(
        onShowPrompt: (GeolocationPermissionsRequestParams request) async {
          final st = await Permission.locationWhenInUse.request();
          if (!st.isGranted) {
            return const GeolocationPermissionsResponse(
              allow: false,
              retain: false,
            );
          }
          return const GeolocationPermissionsResponse(
            allow: true,
            retain: true,
          );
        },
      );
    }
  }

  return controller;
}
```

### 7.3 Normalise paths to `file:` / `content:` URIs (required on Android)

```dart
String filePathToWebViewChooserUri(String path) {
  if (path.isEmpty) return path;
  final lower = path.toLowerCase();
  if (lower.startsWith('content:') || lower.startsWith('file:')) {
    return path;
  }
  return Uri.file(path).toString();
}
```

### 7.4 Implement `setOnShowFileSelector` (camera vs gallery / video)

Return **`List<String>`** of URIs. Empty list = cancel. Use **`FileSelectorParams`** to choose camera vs **`FilePicker`**.

```dart
Future<List<String>> _pickFilesForAndroidWebView(FileSelectorParams params) async {
  final allowMultiple = params.mode == FileSelectorMode.openMultiple;

  final acceptEmpty = params.acceptTypes.isEmpty;
  final wantsVideo =
      params.acceptTypes.any((e) => e.toLowerCase().startsWith('video/'));
  final wantsImage = acceptEmpty ||
      params.acceptTypes.any((e) => e.toLowerCase().startsWith('image/'));
  final wantsPdf = params.acceptTypes.any((e) {
    final l = e.toLowerCase();
    return l.contains('pdf');
  });

  if (params.isCaptureEnabled && wantsImage && !wantsVideo) {
    final st = await Permission.camera.request();
    if (!st.isGranted) return <String>[];

    final xfile = await ImagePicker().pickImage(source: ImageSource.camera);
    if (xfile == null) return <String>[];

    return <String>[filePathToWebViewChooserUri(xfile.path)];
  }

  FileType ft = FileType.any;
  if (wantsImage && wantsVideo) {
    ft = FileType.media;
  } else if (acceptEmpty || wantsPdf) {
    ft = FileType.any;
  } else if (wantsImage) {
    ft = FileType.image;
  } else if (wantsVideo) {
    ft = FileType.video;
  }

  final result = await FilePicker.platform.pickFiles(
    type: ft,
    allowMultiple: allowMultiple,
    withData: false,
  );
  if (result == null || result.files.isEmpty) return <String>[];

  return result.files
      .map((f) => f.path)
      .whereType<String>()
      .where((p) => p.isNotEmpty)
      .map(filePathToWebViewChooserUri)
      .toList();
}
```

### 7.5 Attach the widget and load the URL (avoid race)

Create the controller asynchronously, assign it with **`setState`**, then call **`loadRequest`** on the **next frame** so the Android **`WebView`** is attached before navigation.

```dart
import 'package:flutter/material.dart';
import 'package:webview_flutter/webview_flutter.dart';

class IdentityWebViewPage extends StatefulWidget {
  const IdentityWebViewPage({super.key, required this.initialUri});
  final Uri initialUri;

  @override
  State<IdentityWebViewPage> createState() => _IdentityWebViewPageState();
}

class _IdentityWebViewPageState extends State<IdentityWebViewPage> {
  WebViewController? _controller;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addPostFrameCallback((_) => _bootstrap());
  }

  Future<void> _bootstrap() async {
    final c = await createHostedWebViewController();
    await c.setNavigationDelegate(
      NavigationDelegate(
        onProgress: (p) { /* update progress bar */ },
      ),
    );

    if (!mounted) return;
    setState(() => _controller = c);

    WidgetsBinding.instance.addPostFrameCallback((_) async {
      final wc = _controller;
      if (!mounted || wc == null) return;
      await wc.loadRequest(widget.initialUri);
    });
  }

  @override
  Widget build(BuildContext context) {
    final c = _controller;
    return Scaffold(
      appBar: AppBar(title: const Text('ID Check')),
      body: c == null
          ? const Center(child: CircularProgressIndicator())
          : WebViewWidget(controller: c),
    );
  }
}
```

Optional debugging:

```dart
await controller.setOnConsoleMessage((message) {
  debugPrint('WEB_CONSOLE: ${message.level.name}: ${message.message}');
});
```

### 7.6 Integration checklist

1. **`fromPlatformCreationParams`** + **`onPermissionRequest`** → request OS permission → **`request.grant()`** when appropriate.
2. **`AndroidWebViewController.setOnShowFileSelector`** returns **URI strings**, not raw paths.
3. **`loadRequest`** runs after **`WebViewWidget`** exists (post-frame after **`setState`**).
4. Use **HTTPS** for the hosted URL. If the host rejects default WebView user agents, set a **`User-Agent`** per your integration agreement.

### 7.7 Logging (optional)

Use stable tags in logcat (e.g. **`WEBVIEW_FILE:`**, **`WEBVIEW_JS:`**) to separate native picker issues from JavaScript errors.

---

## 8. Flutter vs other stacks (why Kotlin is not the focus here)

Cleared’s **onboarding** shell and **in-app verification** paths are implemented in **Flutter**. The supported integration is **`webview_flutter`** + **`webview_flutter_android`**, which wraps the same underlying Chromium **`WebChromeClient`** behaviour as Kotlin, but exposes it as **Dart** APIs (`setOnShowFileSelector`, `setGeolocationPermissionsPromptCallbacks`, `onPermissionRequest` on `WebViewController.fromPlatformCreationParams`). **Follow [§7](#7-flutter-implementation-guide)** — that is the canonical recipe aligned with Cleared apps.

If your organisation embeds the same URLs in a **non-Flutter** Android app, you still implement **`onShowFileChooser`**, **`onPermissionRequest`**, geolocation callbacks, and **`file:`/`content:`** URIs against the **Android SDK** directly. That is standard platform documentation (e.g. [WebChromeClient.onShowFileChooser](https://developer.android.com/reference/android/webkit/WebChromeClient#onShowFileChooser(android.webkit.WebView,%20android.webkit.ValueCallback%3Candroid.net.Uri[]%3E,%20android.webkit.WebChromeClient.FileChooserParams)). This guide does **not** duplicate it in Kotlin, to avoid implying Cleared maintains a parallel Kotlin-first path.

---

## 9. iOS notes

- **`WKWebView`** usually handles `<input type="file">` without a custom chooser API.
- **Info.plist usage strings** ([§5.1](#51-layer-1--os-declaration-android-manifest--ios-infoplist)) are mandatory: **`NSCameraUsageDescription`**, **`NSMicrophoneUsageDescription`**, **`NSPhotoLibraryUsageDescription`** (and **`NSPhotoLibraryAddUsageDescription`** if you save back to the library), plus **`NSLocationWhenInUseUsageDescription`** if the hosted page uses geolocation. Without them, the system **never** entitles the app (and thus the WebView) for that sensor.
- **`webview_flutter_wkwebview`:** optional **`WebKitWebViewControllerCreationParams`** for inline media if your pages need it.

---

## 10. Hosted page behaviour (summary)

Typical identity flows in a WebView:

- **`<input type="file">`** with **`accept="image/*"`** or **`video/*`**, sometimes with **`capture`** for direct camera.
- JavaScript listening for **`change`** and reading **`files`**.
- Some steps may rely on **camera capture only**; your **`onShowFileChooser`** / **`setOnShowFileSelector`** implementation should still honour **`capture`** for images when the page requests it.

**Address verification (same WebView host):**

- **`navigator.geolocation`** — On **Android**, register **`setGeolocationPermissionsPromptCallbacks`**, request **location** runtime permission, then return **`GeolocationPermissionsResponse(allow: true, …)`**. Declare **`ACCESS_FINE_LOCATION`** / **`ACCESS_COARSE_LOCATION`** in the manifest. On **iOS**, add **`NSLocationWhenInUseUsageDescription`**.
- **Proof-of-address file** — Prefer an explicit **`accept`** that includes **PDF** (e.g. `image/*` and `application/pdf`) so native pickers can match the page; if **`accept` is empty**, use **`FileType.any`** (or equivalent) for the file chooser so PDFs are not excluded.

Coordinate **environment URLs** (e.g. QA vs production) with Cleared as part of your rollout.

---

## 11. Troubleshooting

| Symptom | Likely cause | Action |
|--------|----------------|--------|
| Tap upload / take photo, nothing happens | No `onShowFileChooser` / `setOnShowFileSelector` | Implement the bridge ([§3](#3-android-required-native-behaviour)) |
| Logcat: `invalid Uri` + raw `/data/...` path | Path returned without `file:` / `content:` | Apply URI normalisation ([§4](#4-the-file-uri-contract-critical)) |
| Camera opens, then page unchanged | URI issue or JS error | Fix URI; inspect WebView console |
| Camera never opens | Permission or intent resolution | Runtime + manifest permissions; `<queries>` |
| Gallery empty / cannot pick | Media permissions on API 33+ | `READ_MEDIA_*` |
| Black in-page preview but file path works | `getUserMedia` not granted | `onPermissionRequest` + HTTPS |

---

## 12. QA checklist

Test on a **physical Android device** (WebView differs from Chrome).

- [ ] **ID front / back (image):** Camera and gallery paths both advance the step.
- [ ] **Selfie (image, often front camera):** Capture completes and the step advances.
- [ ] **Video step (if the flow includes it):** Record or pick video; page validation (e.g. length) passes.
- [ ] **Cancel picker:** No crash; user can retry.
- [ ] **Deny camera permission:** Your app shows a clear message or retry path.
- [ ] **Cold start:** No race where **`loadRequest`** runs before the platform **`WebView`** exists (Flutter: post-frame after **`setState`** with controller).

---

## 13. Security and privacy notes

- Load hosted pages over **HTTPS** (`getUserMedia` and modern file APIs expect a secure context).
- Avoid logging **full paths** or **PII** in production; use redacted diagnostics.
- Define retention and deletion for any cached media under your agreements.
- Prefer **`FileProvider`** with **`exported="false"`** when sharing **`content://`** URIs into WebView on strict devices.

---

## Support

For **session APIs**, **authentication headers**, **allowed origins**, or **hosted URL** details, use your **Cleared integration or technical contact**. This document covers **native WebView integration** only.

---

## Document history

| Version | Date | Notes |
|---------|------|--------|
| 1.0 | 2026-04-27 | First published implementation guide |
| 1.1 | 2026-04-27 | Flutter + Kotlin snippets |
| 1.2 | 2026-04-27 | Client-facing tone; removed internal repository references; retitled sections |
| 1.3 | 2026-04-27 | Address verification: geolocation + PDF / empty-`accept` file picker notes ([§10](#10-hosted-page-behaviour-summary)) |
| 1.4 | 2026-04-27 | Flutter-first guide: removed Kotlin sample block; §8 explains stack choice |
| 1.5 | 2026-04-27 | §5 expanded: manifest, runtime, and WebView permission bridges (“entitling” the WebView) |
