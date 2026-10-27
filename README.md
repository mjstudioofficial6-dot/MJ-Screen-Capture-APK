MJ Screen Capture APK
Project Status
Phase 1 — Clean Android project and build verification.
This repository is for a native Android application being developed and tested from a phone using GitHub and GitHub Actions.
Main Goal
The final app will capture a live video shown on the Android phone screen and monitor two fixed card areas:
Right = first card area
Left = second card area
The app must detect only whether a card is present or absent. It must NOT recognize card number, suit, name, or denomination.
When a new card appears:
Capture one crop image of that area.
Save only one image for that appearance.
Do not save duplicate frames while the same card remains visible.
When the area becomes empty for consecutive frames, re-arm detection.
Right and Left detection work independently.
If both appear at the same time, Right is saved first.
If Left appears first, it must not be missed.
Save sequence follows capture order: 001, 002, 003...
Runtime Requirements
Native Android
Kotlin
XML Android Views
Android MediaProjection
VirtualDisplay
ImageReader
Bitmap/crop processing
Foreground Service for screen capture
Background image processing
SQLite local database
App-private storage
100% offline at runtime
Not allowed in the final runtime:
Firebase
Login/account system
External server
Cloud AI/API
OCR
YOLO
Automatic card-name/suit recognition
Bypass of protected content or Android security restrictions
Detection Logic
The intended lightweight detection state machine is:
EMPTY → CARD CANDIDATE → CARD STABLE → SAVE ONE IMAGE → LOCKED → EMPTY → READY
Detection will use image-change/comparison methods against calibrated empty-area samples.
The system must tolerate small animation or pixel changes and must require sufficient stability before saving.
Area Setup
The user will be able to configure:
Set Right Area
Set Left Area
Calibrate Empty
Test Right
Test Left
Save Areas
Reset Areas
The user draws rectangles on a captured/frozen sample.
Coordinates are stored relative to the captured frame. Resolution and orientation changes must be detected and handled safely.
Data Storage
Captured card crops are stored in app-private storage and are not automatically placed in the phone gallery.
SQLite stores:
Internal ID
Sequence number
Side (Right/Left)
Image path
The displayed sequence must come from database order, not from filenames alone.
History
History will show a grid of saved thumbnails.
Each item can later support:
Open
Edit
Crop
Rotate
Delete
Move
Change/Replace
The initial version should remain simple and should not add unnecessary features.
Reference Section
A separate Reference section will contain 52 manually managed image slots for the standard playing cards.
Reference images are stored privately.
Important: Reference images are NOT used for automatic card recognition in the current version.
Main Screens
Main
Capture Area
Capture / Status
History
Image Detail / Edit
Reference
Basic Settings
Technical Baseline
compileSdk: 36
targetSdk: 36
minSdk: 26
Java/JDK: 17
Android Gradle Plugin: 9.4.0
Gradle: 9.6.0
Kotlin: AGP built-in Kotlin support
XML Android Views
SQLiteOpenHelper
Do not add org.jetbrains.kotlin.android when using AGP 9 built-in Kotlin support.
Development Roadmap
Phase 1
Clean project + successful GitHub Actions build + installable APK.
Phase 2
MediaProjection screen capture.
Phase 3
Capture Area setup and calibration.
Phase 4
Right/Left card presence detection and state machine.
Phase 5
Image saving + SQLite.
Phase 6
History and image management.
Phase 7
52-card Reference section.
Phase 8
Testing with real video:
10 rounds
50 rounds
100 rounds
Build Rules
Development must proceed one phase at a time.
Do not start the next phase until the current phase builds and passes its basic test.
When a GitHub Actions build fails:
Read the exact failed step.
Read the final error lines.
Identify the actual root cause.
Change only the required file(s).
Run the build again.
Do not randomly rewrite the whole project.
Do not claim a build is fixed until GitHub Actions completes successfully.
Important Current Build Note
The repository currently uses GitHub Actions for cloud compilation only.
Cloud is NOT used by the final APK for AI, data storage, card detection, or runtime processing.
The final application must work offline after installation.
Requirement Protection
The original requirements are the baseline.
Do not silently add:
OCR
YOLO
card recognition
cloud AI
Firebase
login
unnecessary analytics
unnecessary permissions
unnecessary background services
unrelated UI/features
Any requirement change must be explicitly discussed before implementation.
Error Handling
When reporting a build problem, always provide:
Failed GitHub Actions step
Exact error message
Root cause
File/path that needs changing
Complete replacement content when a file must be replaced
Exact next build/test steps
Do not guess from warnings when the actual failure is available in the log.
Current Phase 1 Success Condition
Phase 1 is complete only when:
GitHub Actions build is green
Debug APK is generated
APK artifact can be downloaded
APK installs on the target Android phone
Basic Phase 1 screen opens without crashing
Only after these conditions are met should Phase 2 begin.
