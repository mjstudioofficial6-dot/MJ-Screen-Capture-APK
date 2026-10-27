MJ Screen Capture APK

Project Purpose

This repository builds a native Android application that captures a live video shown on the phone screen and saves one crop when a new card appears in either of two fixed screen areas. The app tracks presence and absence only. It must not identify card rank, suit, name, or denomination.

The app is developed and built from a phone using GitHub and GitHub Actions. GitHub Actions is only the build environment; the installed APK must work offline.

Current Status — 28 October 2026

Phase 8 corrective testing is in progress. Do not mark the project complete yet.

Observed on the phone in the RC4 test build:

- MediaProjection starts and the received-frame counter increases.
- Area setup and notification actions open/work.
- The app saves images to History.
- Auto-capture can save duplicate or incorrect images when the video brightness blinks or changes briefly.
- Some saved crops appear blank or include the wrong part of the screen, and the Left/Right preview labels have appeared reversed.
- An empty-background preview showed cards in the selected areas. That sample must not be confirmed as an empty baseline.

RC5 is intended to correct the calibration workflow, crop mapping, and duplicate-save behavior. RC5 is not considered verified until GitHub Actions builds successfully and the installed APK passes the phone tests in this README. Update this section only after a test has actually passed.

Definitive Screen-Area Mapping

Use physical phone-screen positions, not the order in which the areas are configured:

- LEFT = physical left side of the screen = Dragon area.
- RIGHT = physical right side of the screen = Tiger area.

The crop previews and saved History labels must match these definitions. If a preview for LEFT shows the card from the physical right side, calibration/crop mapping is wrong and must be fixed before saving or testing automatic capture.

When a new card appears, save one crop for that appearance. If both areas become eligible on the same analyzed frame, RIGHT is saved first. If LEFT appears first, it must still be captured first. Each side has its own independent detection state.

Required Runtime Behavior

1. Capture the screen through Android "MediaProjection", "VirtualDisplay", and "ImageReader".
2. Monitor only the two saved rectangular regions.
3. Detect card presence/absence through image comparison against a user-confirmed empty-background baseline. Do not use OCR, YOLO, card-symbol templates, or card-name recognition.
4. Save a crop only after a persistent, stable change indicates that a card has appeared.
5. Save exactly one image for that appearance. Keep the region locked while the card remains visible or the result is ambiguous.
6. Re-arm only after that region has matched its empty baseline for consecutive analyzed frames.
7. Ignore ordinary brightness/contrast flashes and small display changes as far as practical. Raw pixel inequality alone must not trigger a save.
8. Never save a blank/background crop as a card image. If the evidence is ambiguous, do not save; keep checking.
9. Save in actual capture order using database sequence numbers: "001", "002", "003", etc. When both sides are ready at the same time, RIGHT takes priority.

Recommended state machine per region:

"EMPTY → CARD CANDIDATE → CARD STABLE → SAVE ONE IMAGE → LOCKED → EMPTY CONFIRMED → READY"

The detection implementation should use illumination-tolerant comparison (for example, grayscale/brightness normalization plus a meaningful changed-area threshold), multi-frame stability checks, and a separate consecutive-empty check. Keep these thresholds in one clearly named configuration area so they can be tuned with real tests. A single brightness pulse or one changed frame must never save an image or re-arm a locked region.

Calibration: Two Different Samples

The full-screen Area Selection Sample and the Empty Background Sample have different jobs. Do not treat them as the same sample.

1. Area Selection Sample

- Capture a frozen full-screen frame through the app's notification action.
- Use this frame only to draw the LEFT and RIGHT rectangles.
- It is not an empty baseline and must never be inserted into History.
- The user must be able to select, adjust, test, reset, and save both rectangles.

2. Empty Background Sample

- Capture a second full-screen frame through the notification action while no card is visible in either selected region.
- After both samples are available, open Capture Area and draw/select LEFT and RIGHT rectangles on the Area Selection Sample.
- Apply those same screen coordinates to the Empty Background Sample and show both resulting crop previews labelled LEFT and RIGHT.
- The previews must be checked before saving. The user confirms that neither crop contains a card, then saves the areas and the empty baselines together.
- One empty-background frame must produce two separate private baseline crops: one for LEFT and one for RIGHT.
- Empty samples and area-selection samples are calibration data only. They must not appear in History or count as captured cards.
- The notification should make the next action clear: after Capture Area Sample, offer Capture Empty Sample. The app should then let the user return to the configuration screen and save both samples together.
- Calibration remains saved between capture sessions until the user changes or resets it. Provide a clear way to recapture/replace the empty baseline in Settings.

Important: The app does not recognize card symbols, so it cannot claim to visually verify that a sample contains no card. The preview and explicit user confirmation are required. If either preview visibly contains a card, do not save it as an empty baseline; capture another empty frame.

Coordinate Safety

- Store region rectangles in the captured frame's pixel coordinate system.
- Use the same mapping transform between the displayed/frozen sample and the actual captured bitmap. Correctly account for "ImageView" scaling, letterboxing/cropping, status/navigation bars, and display size.
- Before saving calibration, test that LEFT preview contains only the physical-left/Dragon region and RIGHT preview contains only the physical-right/Tiger region.
- If resolution, orientation, or capture-frame dimensions change, invalidate or safely remap calibration and ask the user to recalibrate. Never silently reuse invalid coordinates.
- Only pixels inside the relevant selected region may be saved as a card crop.

Capture Controls and Notifications

- Use a foreground service while screen capture is active, with a persistent notification and explicit Stop control.
- The notification should offer Auto Capture Start/Pause, Stop Capture, and the appropriate calibration-sample action.
- Auto Capture must be unavailable until both regions and both valid empty-baseline crops have been saved.
- Starting a session must not silently resume an old or invalid calibration.
- Follow Android MediaProjection consent and lifecycle requirements. Stop/cleanup must release the projection, virtual display, image reader, bitmaps, and service resources.

Storage and History

- Save captured crops in app-private storage; do not automatically add them to the phone gallery.
- SQLite ("SQLiteOpenHelper") stores at minimum: internal ID, sequence number, side ("LEFT"/"RIGHT"), and image path.
- Display sequence from database order, not filename alone.
- History contains only accepted card captures—not area samples, empty baselines, blank frames, or temporary test previews.
- History shows numbered thumbnails and side labels, without card names or dates.
- Support basic open/view and, as already implemented, image export. Edit/crop/rotate/delete/move/replace can be added without changing detection behavior.
- Calibration images and empty baselines stay separate from History.

Main Screens

- Main / Capture Status
- Capture Area and Calibration
- History
- Image Detail / Edit
- Reference (optional manual storage only)
- Basic Settings, including recapture/replace empty-background baseline

The optional 52-card Reference section is for manually managed images only. It must not be used for automatic card recognition in this version.

Offline and Privacy Requirements

The final APK must run fully offline at runtime. Do not add:

- Firebase, login/accounts, analytics, or an external server
- Cloud AI/API calls or network-dependent runtime processing
- OCR, YOLO, or automatic rank/suit/card-name recognition
- Bypasses for protected content or Android security restrictions
- Unnecessary permissions, services, or unrelated UI features

Screen capture must use Android's supported MediaProjection consent flow. If a source video blocks capture through Android security protections, do not attempt to bypass that protection.

Technical Baseline

- Native Android; Kotlin; XML Android Views
- "compileSdk = 36"
- "targetSdk = 36"
- "minSdk = 26"
- JDK 17
- Android Gradle Plugin 9.4.0
- Gradle 9.6.0
- AGP built-in Kotlin support
- "SQLiteOpenHelper" and built-in file APIs

Do not apply "org.jetbrains.kotlin.android" when using AGP 9 built-in Kotlin support.

Build and Error-Fix Rules

When GitHub Actions fails:

1. Identify the exact failed step.
2. Read the first actual compiler/tool error ("e: ...", "error:", or the first causal error), not only the final stack trace.
3. State the root cause and exact file/path.
4. Change only the files required to fix that cause.
5. Re-run the build and inspect the complete result.
6. Do not claim a fix worked until GitHub Actions succeeds. A green build alone does not prove the app works; install and test the APK too.
7. Do not replace unrelated code or silently change requirements.

Keep the project source ZIP and workflow filenames consistent with the actual workflow. Avoid leaving multiple active workflows that build different/old source versions.

Phase 8 Phone Test Checklist

Run these checks on the installed APK and record the result for each:

1. Build succeeds; APK artifact downloads and installs/updates without a crash.
2. Capture Area Sample and Empty Background Sample are separate, and both can be captured from the notification.
3. Select and save LEFT/Dragon and RIGHT/Tiger rectangles; verify preview labels and crops are correct.
4. Capture an empty frame while both regions truly have no card; confirm both empty crops visually before saving.
5. Start Auto Capture with an empty screen; History count must not increase.
6. Show one stable card in LEFT; exactly one correctly cropped "LEFT" image is saved.
7. Show one stable card in RIGHT; exactly one correctly cropped "RIGHT" image is saved.
8. Keep a card visible while brightness/contrast flashes; no duplicate save is allowed.
9. Let the region become empty; it must re-arm only after consecutive empty frames.
10. Show the next card in the same region; exactly one new image is saved.
11. Make LEFT appear first; LEFT must not be missed.
12. Make both appear together; RIGHT is saved first, then LEFT.
13. Check at least 10 rounds, then 50 and 100 rounds. Count expected appearances against History and inspect crop boundaries and side labels.
14. Stop Capture; the notification/service stops cleanly and saved History remains available offline.

Do not declare Phase 8 complete until the build is green and the relevant runtime tests pass. If duplicate/blank/wrong-side crops remain, fix the detection or coordinate mapping before increasing the test count.

Requirement Protection

This README is the requirement baseline. Do not silently add or remove behavior. Discuss any material requirement change before implementation. If this README and code disagree, follow the requirement baseline and correct the implementation rather than weakening the requirement.
