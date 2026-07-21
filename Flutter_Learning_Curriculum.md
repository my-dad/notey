# Flutter Learning Curriculum for an Experienced Android Developer

## Production Note App: Local-First, Tested, and Store-Ready

**Audience:** An experienced native Android developer who already understands Kotlin/Java, Android lifecycles, MVVM/MVI, REST APIs, SQLite/Room, dependency injection, and mobile release processes  
**Core duration:** 8 weeks  
**Schedule:** 40 sessions, approximately 2 hours per session, 5 sessions per week  
**Primary platform:** Android  
**Secondary platform:** iOS, smoke-tested throughout rather than postponed until the end  
**Core capstone:** A polished local-first note-taking app released through a Google Play testing track  
**Optional extension:** Authentication and cross-device cloud synchronization

> This curriculum is optimized for learning Flutter efficiently—not relearning general mobile development from zero.

---

## 1. Curriculum Goals

By the end of the core track, the learner should be able to:

1. Write idiomatic Dart with null safety, immutable models, Futures, Streams, and pattern matching.
2. Build adaptive Flutter interfaces with Material 3.
3. Understand the widget, element, and render-object relationship at a practical level.
4. Manage screen and business state with Cubit/BLoC.
5. Navigate with `go_router`, including route guards and deep links.
6. Consume REST APIs with Dio and serialize models safely.
7. build a reactive local database with Drift.
8. Implement a realistic local-first data flow.
9. Test business logic, widgets, navigation, persistence, and critical end-to-end flows.
10. Profile rebuilds, jank, startup, memory, and network behavior.
11. Prepare Android and iOS builds without delaying platform checks until the final week.
12. release an app through internal or closed testing and understand the path to production.

---

## 2. Scope

### Core Track

The core app includes:

- Create, read, update, archive, restore, and delete notes
- Search, sorting, filtering, and tags
- Draft preservation
- Local persistence with Drift
- Reactive UI updates from database streams
- Material 3 light and dark themes
- Adaptive phone and tablet layouts
- Accessibility and localization foundations
- Local reminders for selected notes
- Import/export or backup to a local file
- Unit, widget, integration, and migration tests
- Android release build and Play testing track
- iOS build and simulator/device smoke tests

### Optional Cloud Track

The optional extension includes:

- Sign up, sign in, sign out, and session restoration
- Access and refresh tokens
- Secure credential storage
- Remote note API
- Local mutation queue
- Retry and idempotency
- Conflict detection and resolution
- Cross-device synchronization
- HTTPS App Links and Universal Links

### Explicit Non-Goals

The core track does **not** require:

- Building a Spring Boot or Ktor backend
- Rewriting the entire app from Provider to BLoC
- Adding analytics merely to tick a checklist
- Implementing exact 15-minute background synchronization
- Supporting Firebase Dynamic Links
- Eliminating all widget-local state
- Chasing an arbitrary test-coverage percentage
- Publishing to both public stores within eight weeks

---

## 3. Primary Technical Choices

| Area | Primary Choice | Reason |
|---|---|---|
| UI | Flutter + Material 3 | Current Flutter default design system |
| Navigation | `go_router` | Declarative routing, redirects, nested navigation |
| State management | Cubit first, BLoC where events add value | Maps well to MVI and avoids unnecessary ceremony |
| Dependency composition | `RepositoryProvider` + `BlocProvider` | One clear composition mechanism |
| Local database | Drift | Strong Room-like mapping, typed queries, reactive streams |
| Networking | Dio | Interceptors, cancellation, multipart, timeouts |
| Serialization | `json_serializable` | Explicit generated model mapping |
| Secure storage | `flutter_secure_storage` | Appropriate for tokens and secrets |
| Testing | `test`, `flutter_test`, `bloc_test`, `integration_test` | Official and ecosystem-standard tools |
| Background work | Best-effort OS scheduling | Honest Android/iOS behavior |
| Deep linking | Android App Links + iOS Universal Links | Standards-based replacement for Dynamic Links |
| Crash reporting | Optional Sentry or Firebase Crashlytics | Add only after privacy and release needs are clear |

### Why Provider Is Not the Main Architecture

Provider is worth understanding because it appears in many Flutter codebases. It will be covered in a comparison exercise, but the capstone will not be built fully in Provider and later rewritten in BLoC. That migration would consume time without increasing the quality of the final app.

### Why Drift Is Preferred Over Raw `sqflite`

A developer familiar with Room benefits more from Drift's typed schema, reactive queries, generated code, migrations, and testability. One raw SQLite exercise is included so the learner understands the abstraction.

---

## 4. Android-to-Flutter Mental Map

| Android | Flutter |
|---|---|
| Activity/Fragment | Route + widget subtree |
| XML layout | Dart widget tree |
| View lifecycle | `State` lifecycle |
| `ViewModel` + `StateFlow` | Cubit/BLoC + immutable state |
| Compose recomposition | Widget rebuild |
| RecyclerView | `ListView.builder`, slivers |
| Room | Drift |
| Retrofit/OkHttp | Dio |
| Kotlin serialization/Moshi | `json_serializable` |
| Hilt composition root | Repository/Bloc providers at app boundaries |
| Navigation Component | `go_router` |
| WorkManager | Flutter plugin over best-effort platform scheduling |
| Espresso/Compose UI test | `flutter_test` and `integration_test` |
| App Links | App Links + Universal Links |
| EncryptedSharedPreferences | `flutter_secure_storage` |

> These are useful mappings, not perfect one-to-one equivalents.

---

## 5. Working Method

Each two-hour session should use approximately:

- **15 minutes:** Read official documentation and inspect the relevant API.
- **75 minutes:** Implement the session task without blindly copying a generated solution.
- **20 minutes:** Test, debug, and inspect behavior in DevTools.
- **10 minutes:** Record lessons, mistakes, and Android-to-Flutter differences.

### Definition of Done for a Session

A session is complete when:

- The code compiles.
- The demonstrated flow works on Android.
- A relevant test or manual verification exists.
- Warnings are understood rather than ignored.
- The learner can explain the Flutter concept in their own words.
- iOS is smoke-tested whenever the session affects plugins, permissions, routing, storage, notifications, or builds.

---

# WEEK 1 — Dart and Flutter Foundations

## Session 1.1 — Dart for Kotlin Developers

### Topics

- `final`, `const`, `late`, and `var`
- Null safety and promotion
- Named and positional parameters
- Collection literals and spread operators
- Records and destructuring
- Pattern matching
- Exhaustive `switch`

### Tasks

1. Create a Flutter project and a separate `playground/` folder.
2. Implement immutable `Note`, `Tag`, and `Folder` domain models.
3. Practice:
   - `map`, `where`, `fold`, and `expand`
   - nullable access and pattern matching
   - records returned from functions
4. Add a `copyWith` method manually.
5. Compare manual models with generated immutable-model approaches, but do not add a generator yet.

### Correct Date-Time Model Example

```dart
class Note {
  const Note({
    required this.id,
    required this.title,
    required this.content,
    required this.createdAt,
    required this.updatedAt,
  });

  final String id;
  final String title;
  final String content;
  final DateTime createdAt;
  final DateTime updatedAt;

  Note copyWith({
    String? title,
    String? content,
    DateTime? updatedAt,
  }) {
    return Note(
      id: id,
      title: title ?? this.title,
      content: content ?? this.content,
      createdAt: createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
    );
  }
}
```

> `DateTime(...)` is not a const constructor. Pass a `DateTime` into a const model, or create the timestamp in a non-const factory/function.

### Checkpoint

The learner can model immutable domain data and explain the differences between Kotlin and Dart null safety.

---

## Session 1.2 — Futures, Streams, and Error Boundaries

### Topics

- `Future<T>`
- `async` and `await`
- `Stream<T>`
- `async*` and `yield`
- Cancellation awareness
- Typed failures versus raw exceptions
- `Future.wait`

### Tasks

1. Build a fake repository that loads notes after a delay.
2. Simulate success, timeout, unauthorized, validation, and unknown failures.
3. Return a stream of note-list changes.
4. Fetch notes and tags concurrently using `Future.wait`.
5. Create a small sealed result type.

### Example

```dart
sealed class LoadResult<T> {
  const LoadResult();
}

final class LoadSuccess<T> extends LoadResult<T> {
  const LoadSuccess(this.value);
  final T value;
}

final class LoadFailure<T> extends LoadResult<T> {
  const LoadFailure(this.message);
  final String message;
}
```

### Checkpoint

The learner can choose between a Future and a Stream and can explain where exceptions should be converted into user-facing failure states.

---

## Session 1.3 — Widget Tree and Lifecycle

### Topics

- `Widget`, `Element`, and `RenderObject`
- `StatelessWidget`
- `StatefulWidget`
- `initState`, `didUpdateWidget`, `didChangeDependencies`, and `dispose`
- Build purity
- Keys
- Hot reload versus hot restart

### Tasks

1. Build a note card as a stateless widget.
2. Build an editor with local draft state.
3. Log lifecycle callbacks and observe navigation behavior.
4. Demonstrate when a key preserves or resets state.
5. Inspect the widget tree in Flutter DevTools.

### Important Rule

Widget-local state is valid for:

- Focus
- Controllers
- Password visibility
- Animation state
- Temporary selection
- Expansion state

Business state should not be forced into `setState`, but `setState` is not inherently bad.

### Checkpoint

The learner can explain what actually rebuilds and why `build()` may run frequently.

---

## Session 1.4 — Layout, Constraints, and Scrolling

### Topics

- Flutter's constraint system
- `Row`, `Column`, `Flex`, `Expanded`, and `Flexible`
- `LayoutBuilder`
- `MediaQuery`
- `ListView.builder`
- Slivers
- Overflow debugging

### Tasks

1. Build phone and tablet note-list layouts.
2. Use `LayoutBuilder` to switch between one- and two-pane layouts.
3. Create a long list with `ListView.builder`.
4. Replace it with a `CustomScrollView` containing slivers.
5. Reproduce and fix one overflow intentionally.

### Checkpoint

The learner can reason from parent constraints to child size instead of treating Flutter like Android XML.

---

## Session 1.5 — Forms, Focus, and First Vertical Slice

### Topics

- `Form` and `GlobalKey<FormState>`
- `TextEditingController`
- Focus traversal
- Validation
- Unsaved draft state
- Returning values from routes

### Tasks

1. Build a create-note form.
2. Validate:
   - non-empty title
   - title length
   - content length
3. Preserve an unsaved draft during temporary navigation.
4. Show a confirmation dialog before discarding changes.
5. Connect list → create → return-created-note using in-memory data.

### Week 1 Deliverable

A multi-screen, in-memory notes prototype that works on Android and launches successfully on iOS.

---

# WEEK 2 — Navigation, Design, and Adaptive UI

## Session 2.1 — Declarative Routing with `go_router`

### Topics

- Route configuration
- Path and query parameters
- Named routes
- Nested navigation
- Redirects
- Error routes

### Tasks

1. Configure routes for:
   - `/notes`
   - `/notes/new`
   - `/notes/:id`
   - `/settings`
2. Add a not-found route.
3. Pass a note ID through the URL rather than passing a full mutable object.
4. Test browser-style back-stack reasoning even if the app is mobile-only.

### Checkpoint

Navigation state is explicit and reproducible.

---

## Session 2.2 — Material 3 and Theme Architecture

### Topics

- `ColorScheme`
- `ThemeData`
- Typography
- Component themes
- Light and dark mode
- Dynamic color as an optional enhancement

### Tasks

1. Define light and dark themes from a seed color.
2. Extract semantic spacing and typography constants.
3. Theme cards, forms, dialogs, snackbars, and navigation.
4. Test high text scaling.
5. Avoid hardcoded colors in feature widgets.

### Checkpoint

The app has consistent visual language and usable dark mode.

---

## Session 2.3 — Adaptive Navigation and Two-Pane UI

### Topics

- Navigation bar versus navigation rail
- Master-detail layout
- Foldable/tablet awareness
- Safe areas
- Keyboard insets

### Tasks

1. Use a bottom navigation bar on narrow screens.
2. Use a navigation rail on wide screens.
3. Show list and detail side by side on tablets.
4. Confirm editor usability with the software keyboard open.
5. Test landscape orientation.

### Checkpoint

The app is not merely stretched phone UI.

---

## Session 2.4 — Gestures, Menus, and Interaction Design

### Topics

- `InkWell` versus `GestureDetector`
- Dismiss actions
- Context menus
- Undo behavior
- Haptics
- Destructive-action confirmation

### Tasks

1. Add swipe-to-archive.
2. Add an undo snackbar.
3. Add long-press multi-selection.
4. Add a note action menu.
5. Ensure gestures do not hide essential functionality.

### Checkpoint

Interactions are discoverable and reversible where appropriate.

---

## Session 2.5 — Accessibility and Localization Foundations

### Topics

- Semantics
- Screen-reader labels
- Focus order
- Contrast
- Text scaling
- Localization generation
- Right-to-left smoke testing

### Tasks

1. Add semantic labels to icon-only actions.
2. Test with TalkBack or VoiceOver.
3. Add English and one additional locale.
4. Test at large font sizes.
5. Check an RTL locale for layout breakage.

### Week 2 Deliverable

A polished, adaptive UI shell with declarative routing, themes, accessibility foundations, and localization scaffolding.

---

# WEEK 3 — State Management and Architecture

## Session 3.1 — State Taxonomy and Cubit

### Topics

- Ephemeral UI state
- Screen state
- Domain state
- Persistence state
- Immutable state
- Cubit lifecycle

### Tasks

1. Identify each existing state value and decide where it belongs.
2. Create `NoteListCubit`.
3. Model:
   - initial
   - loading
   - content
   - empty
   - failure
4. Keep focus and controllers local to widgets.

### Checkpoint

State ownership is deliberate rather than centralized by reflex.

---

## Session 3.2 — Repository Boundaries

### Topics

- Repository contract
- Data source versus repository
- Domain model versus persistence model
- Mapping
- Dependency inversion
- Composition root

### Tasks

1. Define a `NoteRepository` interface.
2. Implement an in-memory repository.
3. Inject it with `RepositoryProvider`.
4. Create the Cubit with `BlocProvider`.
5. Avoid introducing `get_it` in the same architecture.

### Checkpoint

Widgets know about Cubits, Cubits know about repository contracts, and repositories hide data-source details.

---

## Session 3.3 — BLoC for Event-Rich Workflows

### Topics

- When Cubit is sufficient
- When explicit events help
- Event transformers
- Concurrency
- Debounce and restartable search

### Tasks

1. Keep basic note CRUD in a Cubit.
2. Implement search as a BLoC or Cubit with debounce.
3. Prevent stale search results from overwriting newer results.
4. Test rapid query changes.
5. Compare the event-driven version with a direct-method Cubit.

### Checkpoint

The learner chooses Cubit or BLoC based on workflow complexity, not fashion.

---

## Session 3.4 — Side Effects and Navigation

### Topics

- `BlocBuilder`
- `BlocListener`
- `BlocConsumer`
- One-time UI effects
- Navigation after success
- Snackbars and dialogs

### Tasks

1. Save a note through a Cubit.
2. Navigate only after successful persistence.
3. Display failures through a listener.
4. Prevent duplicate navigation on rebuild.
5. Separate render state from one-time effects.

### Checkpoint

Build methods remain declarative and side effects are controlled.

---

## Session 3.5 — Provider Comparison Lab

### Topics

- `ChangeNotifier`
- `Consumer`
- `context.watch`, `read`, and `select`
- Strengths and tradeoffs
- Correct testing approach

### Tasks

1. Build a small settings feature with Provider.
2. Test the `ChangeNotifier` directly.
3. Write one widget test using `ChangeNotifierProvider`.
4. Compare rebuild control with `BlocSelector`.
5. Do **not** use Riverpod's `ProviderContainer` with package Provider.

### Week 3 Deliverable

The app uses Cubit/BLoC as its primary state layer, with clean repository injection and a small Provider comparison module.

---

# WEEK 4 — Networking, Serialization, and Optional Authentication

## Session 4.1 — Dio Client and API Models

### Topics

- Base options
- Connect, send, and receive timeouts
- Interceptors
- Cancellation
- Request IDs
- DTO versus domain model

### Tasks

1. Add Dio.
2. Configure environment-specific base URLs.
3. Add safe debug logging that redacts secrets.
4. Create API DTOs with `json_serializable`.
5. Map DTOs into domain models.

### Current Code-Generation Command

```bash
dart run build_runner build
```

Use watch mode when useful:

```bash
dart run build_runner watch --delete-conflicting-outputs
```

### Checkpoint

Network models do not leak directly into the presentation layer.

---

## Session 4.2 — Failure Mapping and Safe Retry

### Topics

- Dio failure categories
- Timeouts
- Cancellation
- Status-code mapping
- Idempotency
- Exponential backoff with jitter

### Tasks

1. Map Dio failures to typed application failures.
2. Retry only transient and safe operations.
3. Do not retry validation failures or most 4xx responses.
4. Add a request idempotency key for retryable creates if the backend supports it.
5. Test cancellation when leaving a screen.

### Correct Backoff Foundation

```dart
Future<T> retryWithBackoff<T>(
  Future<T> Function() operation, {
  int maxAttempts = 3,
}) async {
  Object? lastError;

  for (var attempt = 0; attempt < maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;

      if (attempt == maxAttempts - 1) {
        rethrow;
      }

      final seconds = 1 << attempt; // 1, 2, 4...
      await Future<void>.delayed(Duration(seconds: seconds));
    }
  }

  throw StateError('Unreachable: $lastError');
}
```

> In Dart, `^` is bitwise XOR, not exponentiation.

### Checkpoint

Retries are intentional and do not accidentally duplicate writes.

---

## Session 4.3 — Mock API and Contract Tests

### Topics

- API contracts
- Mock adapters
- Fixture files
- Pagination
- Contract drift
- Empty and malformed responses

### Tasks

1. Define note API request/response examples.
2. Test list, create, update, and delete responses.
3. Test malformed JSON.
4. Test pagination boundaries.
5. Keep the app usable without a real backend.

### Checkpoint

Flutter learning is not blocked by backend development.

---

## Session 4.4 — Authentication Extension

### Topics

- Access and refresh tokens
- Secure storage
- Session restoration
- Route redirects
- Logout
- Token-expiry behavior

### Tasks

1. Implement auth only if a suitable backend or mock auth service exists.
2. Store secrets in `flutter_secure_storage`.
3. Restore the session on app start.
4. Guard protected routes.
5. Clear local private data according to the product's logout policy.

### Checkpoint

Authentication is a replaceable feature boundary rather than a dependency of every screen.

---

## Session 4.5 — Robust Token Refresh

### Topics

- Single-flight refresh
- Queued interceptors
- Retrying the original request
- Refresh-loop prevention
- Failed-session recovery

### Required Design

1. Detect an eligible 401.
2. Ensure only one refresh request runs.
3. Refresh through a separate client or a path excluded from auth refresh.
4. Update stored credentials.
5. update the original request header.
6. Re-execute with `dio.fetch(requestOptions)`.
7. Complete with `handler.resolve(response)`.
8. Logout only when refresh is conclusively invalid.

### Anti-Pattern

Do not pass `RequestOptions` to `handler.next()`. `next()` continues with an error; it does not execute a request.

### Week 4 Deliverable

A tested network layer operating against a mock or real API. Authentication remains optional for the core local-first release.

---

# WEEK 5 — Drift and Local-First Persistence

## Session 5.1 — Drift Schema and Reactive Queries

### Topics

- Tables
- DAOs
- Generated database code
- Type converters
- Watched queries
- Database isolates when needed

### Tasks

1. Create tables for:
   - notes
   - tags
   - note-tag relations
2. Add timestamps and archived/deleted state.
3. Expose watched note-list queries.
4. Connect database streams to the repository.
5. Replace in-memory persistence.

### Checkpoint

UI updates reactively when database content changes.

---

## Session 5.2 — Transactions and Migrations

### Topics

- Atomic operations
- Schema versions
- Migration strategy
- Migration tests
- Destructive fallback risks

### Tasks

1. Add a new note field in a versioned migration.
2. Write an upgrade test from the previous schema.
3. Archive multiple notes in a transaction.
4. Delete a note and its relations atomically.
5. Export a schema snapshot if using Drift tooling that supports it.

### Checkpoint

Existing user data survives app upgrades.

---

## Session 5.3 — Search, Indexing, and Pagination

### Topics

- Indexed columns
- Full-text search options
- Query plans
- Keyset versus offset pagination
- Result ranking

### Tasks

1. Add indexes for common filters and sort orders.
2. Implement search over title and content.
3. Inspect query plans.
4. Test with thousands of generated notes.
5. Avoid loading every note into memory merely to filter it.

### Checkpoint

Search and lists remain responsive with realistic data volumes.

---

## Session 5.4 — Local-First Repository

### Topics

- Local source of truth
- Optimistic updates
- Pending mutations
- Tombstones
- Server identifiers versus local identifiers
- Clock and ordering concerns

### Tasks

1. Make Drift the UI's source of truth.
2. Write locally before attempting remote synchronization.
3. Track mutation status separately from the note content.
4. Retain failed mutations for retry.
5. Avoid reverting a valid local edit merely because one sync attempt failed.

### Checkpoint

The app is fully useful without network access.

---

## Session 5.5 — Connectivity and Sync Triggers

### Topics

- Connectivity signals versus real internet availability
- Foreground sync
- App-resume sync
- Manual refresh
- Backoff
- Dead-letter handling

### Tasks

1. Treat `connectivity_plus` only as a hint.
2. Always handle actual request failure.
3. Trigger synchronization:
   - after local writes
   - when the app resumes
   - after a connectivity change
   - when the user requests refresh
   - during best-effort background execution
4. Support Wi-Fi, mobile, Ethernet, VPN, and other relevant network types.
5. Keep permanently invalid mutations visible for user correction.

### Week 5 Deliverable

A robust reactive local database and a local-first repository that works independently of network availability.

---

# WEEK 6 — Synchronization, Notifications, and Platform Integration

## Session 6.1 — Synchronization Queue

### Topics

- Mutation types
- Queue ordering
- Idempotency keys
- Retry metadata
- Batch versus sequential sync
- Partial failure

### Tasks

1. Model create, update, archive, restore, and delete mutations.
2. Store attempt count and next-attempt time.
3. Ensure a retried create cannot create duplicate server notes.
4. Process independent mutations safely.
5. Test app termination during synchronization.

### Checkpoint

Pending work survives restarts and can be retried safely.

---

## Session 6.2 — Conflict Resolution

### Topics

- Last-write-wins risks
- Version numbers
- ETags
- Updated-at timestamps
- Field-level merge
- User-assisted resolution

### Tasks

1. Detect a server version mismatch.
2. Preserve both local and remote content during a conflict.
3. Build a conflict-resolution screen.
4. Test simultaneous edits from two clients.
5. Document the chosen conflict policy.

### Checkpoint

Synchronization never silently destroys user writing.

---

## Session 6.3 — Local Notifications

### Topics

- Notification permission
- Android channels
- iOS permission flow
- Scheduling
- Time zones
- Notification taps

### Tasks

1. Allow a reminder to be attached to a note.
2. Request permission contextually.
3. Schedule and cancel reminders.
4. Open the correct note from a notification tap.
5. Test daylight-saving/time-zone behavior where relevant.

### Checkpoint

Reminders behave correctly across app restarts and navigation.

---

## Session 6.4 — Background Work: Realistic Expectations

### Topics

- Android WorkManager
- iOS background task constraints
- Doze and battery optimization
- Best-effort execution
- Foreground fallback

### Tasks

1. Register a best-effort sync task.
2. Add appropriate network constraints.
3. Ensure synchronization also occurs on app resume.
4. Make progress resumable after interruption.
5. Do not promise exact 15-minute execution.

### Correct Product Statement

> The app synchronizes opportunistically after edits, on resume, on manual refresh, when connectivity changes, and during OS-scheduled best-effort background opportunities.

### Checkpoint

Correctness does not depend on an exact background schedule.

---

## Session 6.5 — Deep Links and Shareable Routes

### Topics

- Custom schemes for development
- Android App Links
- iOS Universal Links
- Route restoration
- Authentication redirects
- Post-install limitations

### Tasks

1. Open `/notes/:id` from an HTTPS link.
2. Redirect unauthenticated users and return after login.
3. Configure Android asset links.
4. Configure iOS associated domains.
5. Do not use Firebase Dynamic Links; the service shut down on August 25, 2025.

### Week 6 Deliverable

Optional cloud synchronization is resilient, notifications work, and platform integration respects Android and iOS limitations.

---

# WEEK 7 — Testing, Performance, and Quality

## Session 7.1 — Unit and Cubit/BLoC Tests

### Topics

- Fakes versus mocks
- `bloc_test`
- State sequences
- Failure paths
- Concurrency
- Deterministic time

### Tasks

1. Test list loading and empty state.
2. Test create, update, archive, restore, and delete.
3. Test repository failures.
4. Test debounced search.
5. Test synchronization state transitions.

### Checkpoint

Important business behavior is protected without testing implementation trivia.

---

## Session 7.2 — Database and Migration Tests

### Topics

- In-memory/native test database
- DAO tests
- Migration tests
- Transaction rollback
- Stream assertions

### Tasks

1. Test every critical DAO query.
2. Test migration from the previous release schema.
3. Test rollback on a failed transaction.
4. Test reactive query updates.
5. Test queued mutation persistence after reopening the database.

### Checkpoint

Database changes can ship without gambling with user data.

---

## Session 7.3 — Widget Tests

### Topics

- Pumping widgets
- Finding by semantics and keys
- Form interaction
- Golden tests as an optional tool
- Localization and themes

### Tasks

1. Test note-card rendering.
2. Test editor validation.
3. Test light and dark themes.
4. Test empty, loading, and failure states.
5. Test large text scaling.

### Checkpoint

Critical UI behavior is verified without overusing fragile screenshot assertions.

---

## Session 7.4 — Integration Tests

### Topics

- End-to-end flows
- Real database
- Mock network boundary
- Notifications and deep links
- Test isolation

### Tasks

1. Test create → edit → search → archive → restore.
2. Test app restart with persisted data.
3. Test offline edit followed by sync.
4. Test a deep link into note detail.
5. Test authentication redirect if the cloud extension is enabled.

### Checkpoint

The most important user journeys work as a system.

---

## Session 7.5 — DevTools and Performance

### Topics

- Rebuild profiling
- Frame rendering
- Memory
- Network inspection
- Startup
- Image and list optimization

### Tasks

1. Inspect unexpected rebuilds.
2. Use selectors where they measurably help.
3. Profile a list with thousands of notes.
4. Check memory after repeated navigation.
5. Record a baseline startup and scroll profile.

### Quality Guidance

Do not invent universal pass/fail numbers such as “all screens must launch under two seconds” without defining device class, build mode, data size, and measurement method.

### Week 7 Deliverable

The app has meaningful automated coverage, tested migrations, verified user journeys, and a documented performance baseline.

---

# WEEK 8 — Release Engineering and Store Readiness

## Session 8.1 — Android Release Build

### Topics

- App ID and naming
- Signing
- Build variants
- Versioning
- Obfuscation and symbols
- AAB generation

### Tasks

1. Configure the final application ID.
2. Create and protect the release keystore.
3. Remove debug-only logging and secrets.
4. Build and install a release APK locally.
5. Build the signed AAB.
6. Preserve mapping/debug-symbol files as required.

### Checkpoint

A release build works on a physical Android device.

---

## Session 8.2 — iOS Release Smoke Test

### Topics

- Bundle identifier
- Signing
- Capabilities
- Permissions
- Release mode
- TestFlight preparation

### Tasks

1. Configure the bundle identifier.
2. Verify deployment target against the current Flutter-supported minimum.
3. Run a release build on simulator or device as available.
4. Check storage, notifications, links, and localization.
5. Resolve plugin-specific iOS configuration.

### Current Platform Baseline

As of Flutter 3.44 documentation:

- Android support starts at API 24.
- iOS support starts at iOS 13.

Always verify the currently supported range before release.

### Checkpoint

iOS-specific failures have not been deferred until after Android launch.

---

## Session 8.3 — Privacy, Security, and Policy

### Topics

- Data inventory
- Privacy policy
- Data Safety form
- Permission minimization
- Account deletion where applicable
- Secret handling
- Exported Android components

### Tasks

1. Document every collected, stored, and transmitted data field.
2. Remove unnecessary permissions.
3. Complete a privacy-policy draft.
4. Verify secure token handling if auth is enabled.
5. Check dependency licenses and privacy implications.
6. Validate exported activities, services, and receivers.

### Checkpoint

Store declarations match actual app behavior.

---

## Session 8.4 — Play Testing Track

### Topics

- Internal testing
- Closed testing
- Tester feedback
- Pre-launch reports
- Crash and ANR review
- Production-access requirements

### Tasks

1. Create the Play Console app.
2. Complete the store listing and required declarations.
3. Upload the AAB to internal testing.
4. Fix pre-launch report issues.
5. Move to closed testing when ready.
6. Create a feedback channel and test script.
7. For applicable newer personal accounts, plan for at least 12 continuously opted-in closed testers for 14 days before applying for production access.
8. Verify the latest Play Console requirements because store policies can change.

### Checkpoint

The app is distributed through a legitimate testing track with real feedback.

---

## Session 8.5 — Release Candidate and Retrospective

### Topics

- Release checklist
- Known issues
- Rollout strategy
- Monitoring
- Rollback
- Learning retrospective

### Tasks

1. Run the full automated test suite.
2. Test upgrade from the previous database schema.
3. Complete manual checks on at least one physical Android device.
4. Review crash and ANR data from testers.
5. Document known issues.
6. Create release notes.
7. Decide whether the app is ready for production access or requires another closed-test build.
8. Record:
   - concepts learned
   - mistakes made
   - remaining weak areas
   - next project scope

### Week 8 Deliverable

A release candidate distributed through Play testing, with iOS build readiness, privacy documentation, tested migrations, and a clear path to production.

---

# 6. Optional Extension Modules

These modules are not required for the 40-session core.

## Extension A — Raw SQLite Comparison

- Implement one table with `sqflite`.
- Compare raw SQL, mapping, migrations, and streams with Drift.
- Keep Drift as the capstone database.

## Extension B — `get_it` and `injectable`

- Move composition from widget providers into a service locator.
- Compare ownership, test overrides, and visibility.
- Avoid mixing patterns without a reason.

## Extension C — Image Attachments

- `image_picker`
- Permission handling
- Local file lifecycle
- Compression
- Multipart upload
- Orphan-file cleanup

## Extension D — Push Notifications

- Firebase Cloud Messaging or another provider
- Token registration
- Notification routing
- Foreground/background behavior
- User preferences

## Extension E — Desktop or Web

- Keyboard shortcuts
- Resizable layout
- Web persistence limitations
- Browser URLs
- Desktop file export

## Extension F — App Store/TestFlight Submission

- App Store Connect
- Certificates and profiles
- Privacy manifests
- Screenshots and metadata
- TestFlight groups
- Review feedback

---

# 7. Recommended Project Structure

Feature-first organization is preferred over a large global folder for every technical type.

```text
lib/
├── app/
│   ├── app.dart
│   ├── router.dart
│   └── theme/
├── core/
│   ├── database/
│   ├── network/
│   ├── errors/
│   ├── localization/
│   └── widgets/
├── features/
│   ├── notes/
│   │   ├── data/
│   │   │   ├── local/
│   │   │   ├── remote/
│   │   │   ├── mappers/
│   │   │   └── note_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── note.dart
│   │   │   └── note_repository.dart
│   │   └── presentation/
│   │       ├── bloc/
│   │       ├── pages/
│   │       └── widgets/
│   ├── search/
│   ├── settings/
│   ├── reminders/
│   └── auth/                 # optional
└── main.dart

test/
├── features/
├── core/
├── fixtures/
└── helpers/

integration_test/
└── app_flow_test.dart
```

### Structure Rules

- Do not create a layer merely because a diagram says so.
- Keep feature-specific code inside its feature.
- Keep shared code truly shared.
- Prefer explicit mapping at boundaries.
- Do not expose database rows or API DTOs directly to widgets.
- Avoid one BLoC per tiny widget.
- Do not add a “use case” class that only forwards one repository call unless it provides real policy or orchestration.

---

# 8. Testing Strategy

Testing effort should follow risk, not an arbitrary coverage target.

## Highest Priority

- Database migrations
- Note creation and editing
- Draft preservation
- Search correctness
- Archive/delete behavior
- Synchronization queue
- Conflict handling
- Authentication refresh if enabled
- Deep-link routing
- Notification scheduling and cancellation

## Lower Priority

- Framework behavior
- Trivial getters
- Static widget composition with no logic
- Third-party package internals

## Test Pyramid

1. Many fast unit and database tests
2. Focused widget tests for important states and interactions
3. A small number of stable end-to-end integration tests
4. Manual device testing for permissions, notifications, accessibility, release builds, and store behavior

---

# 9. Release Checklist

Use unchecked boxes until each item is actually verified.

## Functionality

- [ ] Create, read, update, archive, restore, and delete notes
- [ ] Search and filters
- [ ] Draft recovery
- [ ] Dark and light themes
- [ ] Local reminders
- [ ] Data export or backup
- [ ] Offline use
- [ ] Optional synchronization behaves safely

## Data Safety

- [ ] Migration tests pass
- [ ] Destructive migration is not enabled accidentally
- [ ] Failed sync does not discard local data
- [ ] Conflict handling preserves both versions
- [ ] Logout behavior is documented
- [ ] Sensitive values are stored securely

## Quality

- [ ] Critical unit tests pass
- [ ] Database tests pass
- [ ] Widget tests pass
- [ ] Integration flow passes
- [ ] Physical Android device test completed
- [ ] iOS smoke test completed
- [ ] Large text test completed
- [ ] Screen-reader test completed
- [ ] RTL smoke test completed

## Performance

- [ ] Release-mode list scrolling profiled
- [ ] Unexpected rebuilds investigated
- [ ] Startup baseline recorded
- [ ] Memory checked after repeated navigation
- [ ] Large dataset tested
- [ ] Network cancellation tested

## Android Release

- [ ] Current target SDK requirement verified
- [ ] Minimum SDK is compatible with current Flutter support
- [ ] Release signing configured
- [ ] AAB generated
- [ ] Release build installed and tested
- [ ] Permissions reviewed
- [ ] Exported components reviewed
- [ ] Data Safety form matches behavior
- [ ] Privacy policy published
- [ ] Internal test completed
- [ ] Closed-test requirements checked for the account

## iOS Readiness

- [ ] Current Flutter-supported iOS minimum verified
- [ ] Bundle identifier configured
- [ ] Signing configured
- [ ] Required capabilities added
- [ ] Permission descriptions added
- [ ] Release build smoke-tested
- [ ] Universal Links tested if enabled

---

# 10. Common Mistakes This Curriculum Avoids

1. **Using `const DateTime(...)`:** `DateTime` does not provide a const constructor.
2. **Using `2 ^ attempt` for exponential backoff:** `^` is bitwise XOR in Dart.
3. **Testing package Provider with `ProviderContainer`:** that container belongs to Riverpod.
4. **Closing a BLoC fetched from context manually:** the creator/owner should close it.
5. **Using `handler.next(requestOptions)` to retry Dio requests:** retry with `dio.fetch` and resolve the response.
6. **Treating connectivity status as proof of internet:** requests can still fail.
7. **Triggering sync only on mobile data:** Wi-Fi, Ethernet, VPN, and other paths exist.
8. **Promising background work every 15 minutes:** Android and iOS scheduling is best effort.
9. **Using Firebase Dynamic Links:** it shut down on August 25, 2025.
10. **Deferring iOS until the end:** plugin and platform differences should be discovered early.
11. **Rebuilding the same app first in Provider and then in BLoC:** learn both without wasting a full rewrite.
12. **Banning all local widget state:** ephemeral UI state belongs near the widget.
13. **Hardcoding store review time:** review and production-access timelines vary.
14. **Pre-checking a release checklist:** evidence should determine whether a box is checked.
15. **Making a custom backend a prerequisite:** mock contracts keep Flutter progress independent.

---

# 11. Current Reference Notes

The following details were verified for this curriculum in July 2026. Recheck them before release because platform and store policies change.

- Flutter supported platforms:  
  https://docs.flutter.dev/reference/supported-platforms
- Dart `build_runner`:  
  https://dart.dev/tools/build_runner
- Google Play testing requirements for newer personal accounts:  
  https://support.google.com/googleplay/android-developer/answer/14151465
- Google Play test tracks:  
  https://support.google.com/googleplay/android-developer/answer/9845334
- Firebase Dynamic Links shutdown FAQ:  
  https://firebase.google.com/support/dynamic-links-faq
- Flutter BLoC concepts:  
  https://bloclibrary.dev/flutter-bloc-concepts/
- Dio documentation:  
  https://pub.dev/packages/dio
- Drift documentation:  
  https://drift.simonbinder.eu/
- `go_router`:  
  https://pub.dev/packages/go_router
- `connectivity_plus`:  
  https://pub.dev/packages/connectivity_plus

---

# 12. Completion Standard

The curriculum is complete when the learner can:

- Explain Flutter rebuilds and state ownership clearly
- Add a feature without turning the app into a tightly coupled hierarchy
- Change a database schema without losing user data
- Debug an Android and iOS plugin integration
- Handle network failures without corrupting local state
- Write tests around high-risk behavior
- Produce a signed release build
- Distribute the app through an official testing track
- Defend the architecture choices and tradeoffs

The success metric is not “finished 40 sessions.” The success metric is a stable, explainable Flutter app and the ability to build the next one with less guidance.
