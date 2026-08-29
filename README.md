# Multi-Camera Smart Attendance

A Flutter mobile client for a smart classroom attendance system. Students enroll their face, join classes, and review attendance. Teachers create classes, run camera-based attendance sessions, monitor recognition results, and correct attendance records.

The app builds from a single Flutter codebase and runs natively on **both Android and iOS**. It connects to a separate FastAPI backend, which manages authentication, classes, rooms, cameras, face recognition, and attendance calculation.

## Supported platforms

| Platform | Minimum version | Notes |
|---|---|---|
| Android | 5.0 (API 21) | `minSdkVersion` uses the Flutter default (21), which satisfies the CameraX / ML Kit requirement; compiles and targets API 36 with Java 17 |
| iOS | 15.5 | `google_mlkit_face_detection` requires iOS 15.5+, so the deployment target in `ios/Podfile` and the Xcode project is pinned there; runs on iPhone and iPad |
| Web & Backend | refer this > | https://github.com/shubhamxdd/a |

Toolchain notes:

- Building for Android requires the Android SDK (e.g. via Android Studio) and works from any desktop OS.
- Building for iOS requires a Mac with Xcode and CocoaPods. `flutter run`/`flutter build` install the pods automatically; if you change dependencies and pods drift, run `cd ios && pod install`.
- A physical device is recommended for face enrollment on both platforms. Android emulators can emulate a camera feed for general UI/API testing, but the iOS simulator has **no camera**, so enrollment capture requires a real iPhone or iPad.

## Features

### Students

- Register with name, email, roll number, and password
- Complete a guided five-pose face enrollment scan (front, left, right, up, down) using the front camera
- Join classes using a class code
- View attendance percentage and Present/Late/Absent totals, with tap-through lists per status
- Review session history and filter it by date
- Keep the login session and API server setting across app restarts

### Teachers

- Register with an administrator-provided invite code
- Create classes and share their join codes
- View class rosters and individual student attendance
- Start and stop live attendance sessions using a configured room
- Select and preview room camera feeds
- Monitor recent face-recognition sightings
- Review completed sessions
- Override incorrect attendance records (present / late / absent with an optional reason)
- Delete sessions and classes

### Administrators

Administrator accounts can sign in, but room and camera configuration is handled by the backend's web dashboard rather than this mobile client.

## How face attendance works

1. During student registration, the app uses the front camera and Google ML Kit to guide the student through five poses: front, left, right, up, and down.
2. The five JPEG images are uploaded to the backend.
3. The backend validates the images and creates the student's face-recognition profile.
4. A teacher starts a session for a class and a configured room.
5. The backend processes the room's enabled camera feeds, records sightings, and calculates attendance when the session ends.

Face matching is performed by the backend. The mobile app does not use its local TFLite embedding code for production attendance.

## Tech stack

- Flutter and Dart — one codebase for both Android and iOS
- `camera` for enrollment capture (NV21 frames on Android, BGRA8888 on iOS)
- Google ML Kit Face Detection for face and pose guidance
- `permission_handler` for the runtime camera permission
- `http` for REST API communication and multipart image uploads
- `shared_preferences` for token and server URL persistence
- Material 3 UI with Google Fonts (Inter)

Optional demo-only extras (not used for production attendance): `tflite_flutter` + `image` for on-device MobileFaceNet embeddings (`lib/services/embedding_service.dart` and `face_embedder.dart`), `image_picker` for the placeholder profile-photo screen, and `path_provider` for temporary capture files. Drop a `mobilefacenet.tflite` model into `assets/models/` to try them.

## Project structure

```text
lib/
├── main.dart                 # App startup and authentication initialization
├── models/                   # User, class, camera, and attendance models
├── screens/                  # Authentication, dashboards, scans, and sessions
├── services/
│   ├── api_service.dart      # FastAPI REST client
│   ├── auth_store.dart       # Persisted token and API URL
│   └── embedding_service.dart# Optional local embedding/demo support
├── widgets/                  # Shared UI components
└── theme.dart                # App colors and theme
assets/models/                # Optional .tflite embedding model
test/                         # Unit tests for models and URL normalization
```

## Requirements

- Flutter 3.19 or newer
- Dart 3.3 or newer
- Android device or emulator (API 21+) with camera support, or iPhone/iPad on iOS 15.5+
- For iOS builds: a Mac with Xcode and CocoaPods installed
- A running, compatible Smart Attendance FastAPI backend
- At least one backend room and camera configured before starting live sessions

> A physical device is recommended for face enrollment on either platform. An Android emulator can be used for general UI and API testing if it has a usable camera source; the iOS simulator cannot run the enrollment camera at all.

## Getting started

### 1. Install dependencies

From the project root:

```bash
flutter pub get
```

On macOS this also prepares the iOS pods when you next run or build the app. If you change dependencies and see pod-related build errors on iOS, run `cd ios && pod install` manually.

### 2. Configure the backend URL

The URL must point to the machine running the backend. The app automatically appends `/api/v1` when the entered URL does not already contain an API path.

You can configure it in either of these ways:

#### In the app

Open **Server settings** from the sign-in screen and enter a URL such as:

```text
http://192.168.1.10:8000
```

#### At build or run time

```bash
flutter run --dart-define=API_BASE_URL=http://192.168.1.10:8000/api/v1
```

Common development addresses:

| Environment | Example backend URL |
|---|---|
| Android emulator | `http://10.0.2.2:8000` (alias for the host machine's loopback) |
| iOS simulator | `http://127.0.0.1:8000` (the simulator shares your Mac's loopback) |
| Physical Android or iOS device | `http://<computer-LAN-IP>:8000` |
| Production | `https://attendance.example.com` |

The device and backend computer must be reachable from each other. Do not use `localhost` on a physical device—it refers to the phone itself.

On iOS 14 and later, the first request to a local network address triggers the system's local-network permission prompt; allow it so the app can reach the backend.

The source currently has a development fallback URL in `lib/services/api_service.dart`. Prefer the in-app setting or `--dart-define` instead of committing a machine-specific address.

### 3. Run the app

```bash
flutter run
```

To choose a specific device:

```bash
flutter devices
flutter run -d <device-id>        # e.g. an Android emulator or a connected iPhone
```

Release builds:

```bash
flutter build apk --release       # Android APK
flutter build appbundle --release # Android App Bundle (Play Store)
flutter build ios --release       # iOS (requires a signing team configured in Xcode)
```

## Typical workflow

1. An administrator configures rooms and cameras in the web dashboard and provides the teacher invite code.
2. A teacher creates an account and creates a class.
3. Students register, complete face enrollment, and join the class using its join code.
4. The teacher opens the class, starts a live session, enters the configured room code, and selects the attendance settings.
5. The backend processes camera frames while the app displays the live preview and recent sightings.
6. The teacher stops the session, reviews calculated attendance, and applies corrections if required.
7. Students can refresh their dashboards to see the completed session.

## API integration

The client expects a backend under `/api/v1` with endpoints for:

- authentication and student/teacher registration
- current-user validation
- class creation, joining, rosters, and deletion
- room camera discovery
- session creation, live previews, sightings, and stopping
- attendance summaries and teacher overrides

Authentication uses a bearer token returned by the backend. The token and selected API URL are stored locally with `shared_preferences`.

## Permissions and networking

### Android (`android/app/src/main/AndroidManifest.xml`)

The app requests:

- `CAMERA` for student face enrollment
- `INTERNET` for backend communication and live previews

Cleartext HTTP is enabled (`android:usesCleartextTraffic="true"`) for local development, and the ML Kit face-detection model is preloaded at install time via the `com.google.mlkit.vision.DEPENDENCIES` meta-data tag.

### iOS (`ios/Runner/Info.plist`)

The app declares:

- `NSCameraUsageDescription` — capturing face photos for enrollment and verification
- `NSPhotoLibraryUsageDescription` — picking an existing profile photo from the photo library
- `NSLocalNetworkUsageDescription` — connecting to the attendance server on the local network
- `NSAppTransportSecurity` with `NSAllowsArbitraryLoads` — allows plain-HTTP backends during development

Both platforms currently permit cleartext HTTP for development convenience. Use HTTPS in production and apply stricter network security rules (Android network security config, iOS ATS exceptions) before release.

## Platform notes

- The guided enrollment scan (`lib/screens/enrollment_scan_screen.dart`) requests NV21 camera frames on Android and BGRA8888 frames on iOS, with a YUV-420 three-plane fallback, then encodes upright JPEGs in a background isolate. All other logic is shared Dart.
- The front camera is preferred for enrollment; the first available camera is used as a fallback.
- The launcher display names currently differ per platform: Android shows "Atten" (`android:label` in the manifest) while iOS shows "Multi Cam Attendance System" (`CFBundleDisplayName` in Info.plist). Align them if you need consistent branding.

## Testing and analysis

```bash
flutter analyze
flutter test
```

The test suite covers the attendance/user/class data models and the API base-URL normalization logic; it avoids plugins and network access so it runs quickly and deterministically on any machine.

## Privacy and security

Face images and embeddings are biometric data. Before deploying this project:

- obtain informed consent from users
- use HTTPS for all client-server traffic
- restrict access to enrollment images and embeddings
- define retention and deletion policies
- protect backend credentials, tokens, room codes, and teacher invite codes
- follow applicable privacy, biometric-data, and education-record regulations
- never commit production secrets or biometric data to the repository

## Current scope

This repository contains the Flutter client only. The FastAPI backend, recognition workers, database, camera services, and administrator web dashboard must be deployed separately.
