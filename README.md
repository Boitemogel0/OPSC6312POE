# BoiTebogo's Innovative Hub — OPSC6312 Part 2 Prototype

**Student:** Boitemogelo Tseke  
**Student number:** ST10448828  
**Module:** OPSC6312 — Open Source Coding (Intermediate)  
**Application:** BoiTebogo's Innovative Hub  
**Focus:** Centurion, Gauteng, South Africa  
**Tagline:** *Empower. Develop. Succeed.*  
**GitHub target repository:** https://github.com/Boitemogel0/OPSC6312POE.git

## 1. Prototype purpose

BoiTebogo's Innovative Hub is a native Android youth-development prototype that connects talent discovery, skills development, evidence creation and opportunity discovery. The Part 2 implementation concentrates on the assessable prototype journey:

1. Secure Firebase email/password registration and sign-in.
2. Persistent Settings preferences.
3. Authenticated REST/JSON communication.
4. Talent Passport + QR verification.
5. Gift-to-Opportunity matching.
6. Portfolio evidence submission.
7. AccuWeather location intelligence for the Centurion hub context.
8. Validation, loading, empty and failure states.
9. GitHub Actions build and automated tests.

SSO, full offline synchronisation, real-time push notifications and complete multilingual resource delivery are intentionally treated as later/final-POE extensions rather than falsely claimed Part 2 completions.

## 2. Technology stack

### Android
- Kotlin
- Android Studio
- Jetpack Compose + Material 3
- MVVM-style state holders / ViewModels
- Retrofit + OkHttp + Gson
- Firebase Authentication
- Google Code Scanner for QR scanning

### Backend
- Node.js
- Express
- MongoDB Atlas + Mongoose
- Firebase Admin SDK for optional server-side token verification
- HTTPS-ready REST API

### External REST resource
The prototype uses the AccuWeather Locations endpoint supplied for the project:

`https://dataservice.accuweather.com/locations/v1/{locationKey}`

The default Centurion location key used by the prototype is **298073**. AccuWeather documents location keys as unique identifiers used by its Locations API and other weather services. The API requires authenticated requests and the API key must not be committed to Git. See the official documentation: https://developer.accuweather.com/documentation/location-keys and https://developer.accuweather.com/documentation/authentication.

## 3. Architecture

```text
                         BoiTebogo Android App

 User
  |
  v
 Jetpack Compose UI
  |
  v
 ViewModel / UI State
  |
  v
 Repository
  |
  +--------------------+---------------------+
  |                    |                     |
  v                    v                     v
Firebase Auth      Retrofit/HTTPS       Local Settings
  |                    |
  |                    v
  |             BoiTebogo REST API
  |                    |
  |          +---------+----------+
  |          |                    |
  |          v                    v
  |       MongoDB Atlas       AccuWeather
  |                           Locations API
  |
  +--> Firebase ID token used as authenticated API bearer token
```

The UI does not call the network service directly. Network operations pass through the repository boundary, which keeps the prototype testable and maintainable.

## 4. Part 2 user-defined features

### Feature 1 — Talent Passport + QR verification

A user creates a structured talent profile. The backend generates a random revocable token. The QR code contains the token only; it does not contain a password or other sensitive personal information. A second scan calls the verification endpoint and returns a valid/invalid result.

Endpoints:
- `POST /api/v1/passports`
- `GET /api/v1/passports/{token}/verify`

### Feature 2 — Gift-to-Opportunity Engine

The user enters skills/gifts and a preferred location. The backend compares declared skills with stored opportunity skill tags and calculates an explainable score:

- skill overlap: up to 80 points
- location alignment: up to 20 points
- maximum: 100 points

Endpoint:
- `POST /api/v1/match`

### Feature 3 — Portfolio Evidence

The user records evidence metadata such as title, description, category, evidence type and optional URL. The backend associates the record with the authenticated Firebase user ID.

Endpoints:
- `POST /api/v1/portfolio`
- `GET /api/v1/portfolio/{userId}`

## 5. AccuWeather integration

The app Home screen displays a Hub Location Intelligence card. It retrieves Centurion location information through the BoiTebogo REST API proxy:

`GET /api/v1/location/298073`

The backend then calls:

`GET https://dataservice.accuweather.com/locations/v1/298073?language=en-us&details=true`

The server supplies the AccuWeather bearer credential, preventing the application source code from containing the provider key.

Returned information includes location name, country, administrative area, timezone and geographic coordinates where supplied by AccuWeather.

## 6A. Production Firebase + Render deployment

The repository now includes `render.yaml` and `docs/FIREBASE_RENDER_SETUP.md` for the hosted Part 2 environment. The Android Retrofit `BASE_URL` is read from the ignored `android/local.properties` file using `baseUrl=https://<your-service>.onrender.com/`.

The backend is designed for `AUTH_REQUIRED=true` in the hosted environment. Firebase Admin verifies Firebase ID tokens before protected REST operations. The Android client obtains the signed-in Firebase user's ID token and sends it as a Bearer token.

Real project-specific files and credentials still have to be supplied by the project owner: `android/app/google-services.json`, Firebase Admin service-account credentials, MongoDB Atlas URI, AccuWeather API key and Adzuna credentials. These are intentionally not included in this coding pack.

See `docs/FIREBASE_RENDER_SETUP.md` for the exact sequence.

## 6. Firebase setup

1. Create a Firebase project.
2. Register an Android app using package name:

`za.co.boitebogo.innovativehub`

3. Download `google-services.json`.
4. Copy it to:

`android/app/google-services.json`

5. Enable **Authentication → Sign-in method → Email/Password**.
6. Configure a suitable Firebase password policy.
7. Do not commit the real `google-services.json` to Git.

Firebase documentation: https://firebase.google.com/docs/auth/android/password-auth

## 7. Backend setup

From the `backend` directory:

```bash
npm install
copy .env.example .env
npm test
npm start
```

On Linux/macOS use `cp .env.example .env` instead of `copy`.

Required `.env` values:

```text
PORT=8080
MONGODB_URI=mongodb+srv://USERNAME:PASSWORD@CLUSTER.mongodb.net/boitebogo_hub
AUTH_REQUIRED=false
FIREBASE_PROJECT_ID=your-project-id
ACCUWEATHER_API_KEY=YOUR_REAL_KEY
```

For a submitted hosted deployment, use HTTPS and set `AUTH_REQUIRED=true` after configuring Firebase Admin credentials securely in the hosting environment.

## 8. Android API configuration

Before running the Android application, change:

`android/.../data/Network.kt`

```kotlin
private const val BASE_URL = "https://YOUR-HOSTED-API.example.com/"
```

to the real HTTPS address of the deployed BoiTebogo API.

For local testing on an Android emulator, a common local-server address is:

`http://10.0.2.2:8080/`

For a physical phone, use a reachable HTTPS deployment or the development computer's LAN address while the phone and computer are on the same network.

## 9. Android Studio procedure

1. Open Android Studio.
2. Select **Open** and choose the `android` folder.
3. Allow Gradle synchronisation to complete.
4. Add the real `google-services.json`.
5. Update the REST `BASE_URL`.
6. Connect a physical Android device with USB debugging enabled or use an emulator.
7. Select the `app` run configuration.
8. Build and run.
9. Register a test user.
10. Demonstrate the complete Part 2 workflow.

## 10. Suggested demonstration account

Create a fresh Firebase account specifically for the demonstration. Do not place a real password in this README or Git history.

Suggested demo inputs:

```text
Display name: BoiTebogo Demo User
Skills: Android, Kotlin, Creativity, Communication
Location: Centurion
Portfolio title: Youth Innovation Android Prototype
Category: Technology
Evidence type: Project
```

## 11. API endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/health` | API health check |
| GET | `/api/v1/location/{locationKey}` | AccuWeather location proxy |
| GET | `/api/v1/opportunities` | Stored opportunity feed |
| POST | `/api/v1/match` | Explainable skill/location matching |
| POST | `/api/v1/passports` | Create Talent Passport |
| GET | `/api/v1/passports/{token}/verify` | Verify QR token |
| POST | `/api/v1/portfolio` | Submit evidence |
| GET | `/api/v1/portfolio/{userId}` | Retrieve user's evidence |
| GET | `/api/v1/external/jobs` | Optional Adzuna server-side proxy |

## 12. Security notes

- Passwords are handled by Firebase Authentication and are not stored as plaintext in the Android application.
- AccuWeather API credentials belong on the server, not in Git.
- Firebase ID tokens are used as bearer credentials for protected REST endpoints when server authentication is enabled.
- QR codes contain revocable random references rather than sensitive profile data.
- Secrets, service accounts and local configuration are excluded through `.gitignore`.

## 13. Automated testing

Backend tests:

```bash
cd backend
npm test
```

Android unit tests:

```bash
cd android
gradle testDebugUnitTest
```

Build:

```bash
gradle assembleDebug
```

GitHub Actions workflow is located at:

`.github/workflows/android.yml`

The workflow is intended to execute tests and assemble a debug APK on pushes/pull requests.

## 14. Rubric evidence checklist

| Part 2 criterion | Evidence to capture |
|---|---|
| App runs on mobile | Physical-phone launch |
| Sign in | Registration, login, invalid credentials |
| Settings | Toggle persistence after restart |
| REST API creation/use | Backend source + endpoint response |
| REST integration | Android Retrofit request + UI result |
| User Defined 1 | Generate QR + scan/verify |
| User Defined 2 | Skills + location + match scores |
| User Defined 3 | Submit evidence + status |
| UI | Full screen walkthrough |
| GitHub/README/testing | Repository, README and successful Actions run |
| Demonstration video | Professional voice-over walkthrough |

## 15. Demonstration storyboard

1. 0:00–0:30 — App identity and purpose.
2. 0:30–1:30 — Register and demonstrate password validation.
3. 1:30–2:15 — Sign in and Settings.
4. 2:15–3:00 — Home location intelligence and REST data.
5. 3:00–4:00 — Talent Passport and QR verification.
6. 4:00–5:00 — Gift-to-Opportunity matching.
7. 5:00–6:00 — Portfolio evidence.
8. 6:00–6:45 — Invalid input/API error handling.
9. 6:45–7:30 — GitHub, README and Actions.
10. 7:30–8:00 — Close with Part 2 scope and deferred final-POE extensions.

## 16. AI-use declaration

If generative AI was used during development, disclose it according to the institution's academic-integrity requirements. The student remains responsible for understanding, testing, documenting and being able to explain every submitted component.

## 17. Academic references used for implementation

Android Developers (2026) *Compose UI architecture*. Available at: https://developer.android.com/develop/ui/compose/architecture (Accessed: 22 September 2026).

AccuWeather Developer (2026) *Location keys*. Available at: https://developer.accuweather.com/documentation/location-keys (Accessed: 22 September 2026).

AccuWeather Developer (2026) *Authentication*. Available at: https://developer.accuweather.com/documentation/authentication (Accessed: 22 September 2026).

Firebase (2026) *Password authentication on Android*. Available at: https://firebase.google.com/docs/auth/android/password-auth (Accessed: 22 September 2026).

Google Developers (2026) *Google Code Scanner for Android*. Available at: https://developers.google.com/ml-kit/vision/barcode-scanning/code-scanner (Accessed: 22 September 2026).
