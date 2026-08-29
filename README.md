# Multi-Camera Smart Attendance

A Flutter mobile client for a smart classroom attendance system. Students enroll their face, join classes, and review attendance. Teachers create classes, run camera-based attendance sessions, monitor recognition results, and correct attendance records.

The app connects to a separate FastAPI backend, which manages authentication, classes, rooms, cameras, face recognition, and attendance calculation.

## Features

### Students

- Register with name, email, roll number, and password
- Complete a guided five-pose face enrollment scan
- Join classes using a class code
- View attendance percentage and Present/Late/Absent totals
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
- Override incorrect attendance records
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

- Flutter and Dart
- `camera` for enrollment capture
- Google ML Kit Face Detection for face and pose guidance
- `http` for REST API communication and multipart image uploads
- `shared_preferences` for token and server URL persistence
- Material 3 UI with Google Fonts

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
```

## Requirements

- Flutter 3.19 or newer
- Dart 3.3 or newer
- Android device or emulator with camera support
- A running, compatible Smart Attendance FastAPI backend
- At least one backend room and camera configured before starting live sessions

> A physical device is recommended for face enrollment. An emulator can be used for general UI and API testing if it has a usable camera source.

## Getting started

### 1. Install dependencies

From the project root:

```bash
flutter pub get
```

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
| Android emulator | `http://10.0.2.2:8000` |
| Physical Android device | `http://<computer-LAN-IP>:8000` |
| Production | `https://attendance.example.com` |

The device and backend computer must be reachable from each other. Do not use `localhost` on a physical device—it refers to the phone itself.

The source currently has a development fallback URL in `lib/services/api_service.dart`. Prefer the in-app setting or `--dart-define` instead of committing a machine-specific address.

### 3. Run the app

```bash
flutter run
```

To choose a specific device:

```bash
flutter devices
flutter run -d <device-id>
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

The Android app requests:

- `CAMERA` for student face enrollment
- `INTERNET` for backend communication and live previews

Cleartext HTTP is enabled in the Android manifest for local development. Use HTTPS in production and apply stricter network security rules before release.

## Testing and analysis

```bash
flutter analyze
flutter test
```

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
