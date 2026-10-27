MJ Screen Capture APK — Phase 8 RC6 Candidate
Project purpose
A native Android application that captures a live video displayed on the phone screen and saves a crop when a new card appears in one of two fixed screen regions. The app tracks card presence and absence only. It must not identify card rank, suit, name, or denomination.
The project is built from an Android phone using GitHub and GitHub Actions. GitHub Actions is used only to compile the APK. The installed app must work fully offline.
Current status — 28 October 2026
RC5 detection behavior is user-tested and confirmed as the known-good baseline. The user reports that a new card appearance is saved, while brightness blinking and unrelated visual changes do not create extra saves. History showed sequential examples such as 001 through 010.
RC6 is a candidate build for Deck folders and compact History. Source/ZIP/static checks have passed, but RC6's actual Gradle build and phone tests have not yet been confirmed. Do not call RC6 complete until GitHub Actions builds successfully and the updated APK passes the phone tests below.
RC6 must preserve the RC5 DetectionEngine.kt behavior. Do not change the detector, stability thresholds, or re-arm logic unless a reproducible runtime bug is demonstrated.
Definitive screen-area mapping
Use physical phone-screen position, not the order in which the rectangles were configured:
LEFT = physical left side of the phone screen = Dragon region.
RIGHT = physical right side of the phone screen = Tiger region.
The selected-area previews, saved crop images, and History labels must follow this mapping. Verify crop boundaries before accepting calibration.
Each region is monitored independently. If LEFT appears first, it must not be missed. If both regions become eligible in the same analyzed frame, RIGHT is saved first. Otherwise, save in actual accepted-capture order.
Required detection behavior
Capture the phone screen through Android MediaProjection, VirtualDisplay, and ImageReader.
Monitor only the two user-selected rectangles.
Compare the selected areas against a user-confirmed empty-background baseline. Do not use OCR, YOLO, card templates, or card-name/suit recognition.
Require a persistent, stable presence change before saving a card crop.
Save exactly one image per card appearance. Keep that region locked while the same card remains visible or the evidence is ambiguous.
Re-arm only after the region matches its empty baseline for consecutive analyzed frames.
Ignore ordinary brightness/contrast flashes and unrelated small display changes as far as practical. Raw pixel inequality alone must not trigger a save.
Do not save blank/background crops or calibration samples as card images.
Save actual accepted capture order in SQLite, with a sequence starting at 001 for each Deck.
Expected state machine per region:
EMPTY → CARD CANDIDATE → CARD STABLE → SAVE ONE IMAGE → LOCKED → EMPTY CONFIRMED → READY
The RC5 detector behavior is currently the known-good baseline. Do not replace it with a new algorithm without evidence from a reproduced failure.
Calibration: two distinct samples
The app uses two different samples. They must not be confused.
1. Area Selection Sample
A frozen full-screen frame captured through the app's notification action.
Used only to draw/edit the LEFT and RIGHT rectangles.
Must never become an empty baseline or appear in History.
2. Empty Background Sample
A separate full-screen frame captured while no card is visible in either selected region.
Apply the saved rectangles to this frame and show two crop previews labelled LEFT and RIGHT.
The user must visually check both previews and confirm that no card is present before saving the samples as empty baselines.
Store one private baseline crop per region; never insert either baseline into a Deck or History.
The notification workflow should allow the user to capture the Area Selection Sample and then capture the Empty Background Sample. The user can return to area setup, select the two rectangles, preview both empty crops, and save them together.
Settings must provide a way to replace the Empty Background Sample.
The app does not recognize card symbols, so it cannot automatically prove a sample is empty. User confirmation is required. If either preview contains a card, capture a new empty frame.
Coordinate safety
Store selected rectangles relative to the actual captured frame's pixel coordinates.
Correctly map coordinates between the displayed sample and captured bitmap, including ImageView scaling, letterboxing/cropping, status/navigation bars, and display dimensions.
Confirm the LEFT and RIGHT previews show only their intended physical screen regions.
Detect screen-size, orientation, or captured-frame dimension changes. Do not silently reuse invalid coordinates; safely remap or ask for recalibration.
Only pixels inside the appropriate selected rectangle may be saved as the card crop.
Deck folders and capture sessions — RC6
A Deck is a logical folder displayed inside the app and backed by a private app-storage directory. It is not automatically created in the phone's public gallery.
Every new Screen Capture session creates a fresh Deck by default: Deck 1, Deck 2, Deck 3, and so on.
Accepted card crops from that capture session are stored in its active Deck.
Pause Auto Capture pauses detection/saving but keeps the same active Deck. Resume Auto Capture continues in that Deck.
Stop Capture ends the current screen-capture session. The next fresh Screen Capture start creates a new Deck by default.
The user can choose Use Next on an existing Deck to append captures from the next session to that Deck. The chosen Deck must have space under the 400-image limit.
The History Deck list must support Open, Rename, Delete, and Use Next. The user must also be able to choose a new Deck for the next session rather than reusing a previously selected one.
Deck names must be unique. Renaming a Deck changes its displayed name/folder label; deleting Deck 1 must not silently rename the remaining Decks. The user can manually rename another Deck to the freed name.
Deleting a Deck requires confirmation and permanently deletes that Deck's records and images. The currently active Deck cannot be deleted. If the deleted Deck was queued for reuse, clear that selection.
Each Deck has its own sequence numbers, starting at 001. Sequence order is based on database order, not filenames alone.
Each Deck has a hard maximum of 400 images. At 400/400, Auto Capture must pause or stop saving with a clear status message; image 401 must not be added.
Existing RC5 captures must be preserved by the database migration and shown under Previous captures.
History and image management — RC6
Opening History shows the Deck folder list. Opening a Deck shows only that Deck's images.
The selected Deck's History grid uses 10 compact thumbnails per horizontal row. Each thumbnail displays sequence and side, such as 001 L or 001 R.
Tapping a thumbnail opens the larger image/detail view.
Search by sequence or side within the selected Deck.
Preserve image detail/view and any existing crop, rotate, delete, move, replace, and export behavior unless a specific regression is reproduced.
Export This Deck (ZIP) includes only the selected Deck. Export All groups images by Deck in the ZIP and includes a CSV index.
History contains accepted card crops only—not area-selection samples, empty baselines, blank frames, or temporary previews.
Images and calibration assets stay in app-private storage and are not automatically added to the public gallery.
Main screens
Main / Capture Status
Capture Area and Calibration
History / Deck list
Image Detail / Edit
Reference (optional manual storage only)
Basic Settings, including replace-empty-baseline workflow
The optional Reference section contains 52 manually managed standard playing-card image slots. Reference images are not used for automatic recognition.
Capture controls and Android lifecycle
A foreground service operates while screen capture is active, with a persistent notification and explicit Stop control.
Notification controls should include Start/Pause Auto Capture, Stop Capture, and the appropriate calibration-sample action.
Auto Capture must not start until both regions and valid empty-baseline crops have been saved.
Pausing auto-capture must not end the active Deck. Stopping screen capture ends that Deck session.
Follow Android MediaProjection consent/lifecycle requirements and release the projection, virtual display, image reader, bitmaps, and service resources when stopping.
Runtime privacy and offline restrictions
The final APK must work fully offline. Do not add:
INTERNET permission, Firebase, login/accounts, analytics, or an external server
Cloud AI/API or network-dependent runtime processing
OCR, YOLO, or automatic rank/suit/card-name recognition
Any bypass of Android protected-content/security restrictions
Unnecessary permissions, background services, or unrelated features
Use Android's supported MediaProjection consent flow. If a source video blocks screen capture through Android security protections, do not try to bypass that protection.
Technical baseline
Native Android; Kotlin; XML Android Views
compileSdk = 36
targetSdk = 36
minSdk = 26
JDK 17
Android Gradle Plugin 9.4.0
Gradle 9.6.0
AGP 9 built-in Kotlin support
SQLiteOpenHelper and built-in file APIs
Do not apply org.jetbrains.kotlin.android when using AGP 9 built-in Kotlin support. The RC5-to-RC6 database migration must be additive and preserve existing captures.
Build and error-fix rules
When GitHub Actions fails:
Identify the exact failed workflow step.
Read the first actual compiler/tool error (e: ..., error: ...), not only the final stack trace or unrelated warnings.
Identify the root cause and exact file/path.
Change only the required file(s). Do not randomly rewrite the project or touch the working detector without a reproduced issue.
Re-run static checks and the build, then inspect the result.
Do not claim a fix worked until GitHub Actions finishes green. A green build alone does not prove runtime behavior; install and test the APK too.
Ensure the workflow uses the exact source ZIP name MJ_Phase8_Integrated_RC6.zip. Disable conflicting active workflows that might build an older source version.
Preserve the last known-good source ZIP and workflow before risky changes.
Verify artifact download links against the exact generated file paths. If a link fails, recreate the artifact at a new verified path rather than reusing an unverified link.
RC6 phone test checklist
Record each result as pass/fail; do not mark untested items complete.
GitHub Actions static checks pass; :app:assembleDebug succeeds; APK artifact downloads and installs/updates without a crash.
Existing RC5 history remains after the update and appears under Previous captures.
Start Screen Capture; a new Deck appears by default.
Start Auto Capture and introduce cards; each accepted crop is saved in the active Deck with sequences 001, 002, etc.
Pause Auto Capture; no new images are saved. Resume; saving continues in the same Deck.
Stop Capture, then start a new Screen Capture session; a new Deck is created by default.
Select Use Next on a non-full Deck; the next session appends captures to that Deck.
Rename a Deck; its new name appears, duplicate names are rejected, and the user can manually rename a Deck to a name freed by deletion.
Delete a non-active Deck after confirmation; its images/records disappear. Deleting the active Deck is blocked.
At 400 images in one Deck, Auto Capture pauses/reports the limit and image 401 is not added.
A Deck grid displays 10 compact thumbnails per row; tapping a thumbnail opens the full image/detail.
Search by sequence and side works inside the selected Deck.
Export This Deck contains only that Deck; Export All preserves per-Deck folders and its CSV index.
Re-test card changes and brightness blinking to confirm the known-good RC5 detector behavior remains intact.
Expand the test to 10, then 50 and 100 rounds as practical. Compare expected appearances to History and inspect crops, labels, ordering, and duplicate counts.
Stop capture; notification/service stops cleanly and saved data remains available offline.
Requirement protection
This README is the requirement baseline. Do not silently add/remove behavior, networking, recognition capability, permissions, services, or unrelated UI. Discuss material requirement changes before implementation. If code and README conflict, follow this specification and correct the implementation rather than weakening the requirements.
