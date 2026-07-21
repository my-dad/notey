# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

This is currently a bare Flutter scaffold (the stock counter-app template, `lib/main.dart`, with a decrement button added). No curriculum session work has landed in `lib/` yet — it is not yet the note-taking app described below. Treat any architecture described here as the intended target, not the current state, and check `lib/` before assuming a structure exists.

`Flutter_Learning_Curriculum.md` at the repo root is the authoritative design/roadmap doc for this project: an 8-week, 40-session plan (5 sessions/week, ~2h each) to build a local-first note-taking app ("notey" / `simply_notes`) as a learning exercise for an experienced Android developer moving to Flutter. Read it before making architectural decisions — it specifies the intended tech stack, project structure, testing strategy, and a list of specific mistakes to avoid. Key points pulled from it:

### Working method (when doing curriculum session work)
Each session follows: read docs (~15 min) → implement the task by hand rather than pasting a generated solution (~75 min) → test/debug in DevTools (~20 min) → note lessons and Android↔Flutter differences (~10 min). A session isn't done until: it compiles, the flow works on Android, a test or manual verification exists, warnings are understood (not ignored), and iOS is smoke-tested if the session touched plugins, permissions, routing, storage, or notifications. Primary platform is Android; iOS is smoke-tested throughout, not deferred to the end.

### Explicit non-goals (don't do these even if they seem like natural improvements)
- No custom backend (Spring Boot/Ktor) — mock contracts keep Flutter progress unblocked.
- Don't fully rewrite the app from Provider to BLoC — Provider is covered only as a small comparison module; BLoC/Cubit is the real architecture from the start.
- Don't add analytics just to check a box.
- Don't promise exact-interval background sync (e.g. "every 15 minutes").
- Don't chase an arbitrary test-coverage percentage — test by risk (see Testing priorities below).
- Don't eliminate all widget-local `setState` — it's correct for ephemeral UI state (focus, controllers, animation, expansion, selection); only business/domain state belongs in Cubit/BLoC.

### Intended tech stack (not yet in `pubspec.yaml`)
| Area | Choice |
|---|---|
| Navigation | `go_router` |
| State management | Cubit first, BLoC where events add value (not Provider, not Riverpod) |
| DI/composition | `RepositoryProvider` + `BlocProvider` (no `get_it` in the core architecture) |
| Local database | Drift (reactive, typed — not raw `sqflite`) |
| Networking | Dio |
| Serialization | `json_serializable` |
| Secure storage | `flutter_secure_storage` |
| Testing | `test`, `flutter_test`, `bloc_test`, `integration_test` |

### Intended project structure (feature-first)
```
lib/
├── app/            # app.dart, router.dart, theme/
├── core/           # database/, network/, errors/, localization/, widgets/ (shared only)
└── features/
    └── <feature>/
        ├── data/           # local/, remote/, mappers/, *_repository_impl.dart
        ├── domain/         # entities + repository interfaces (no framework deps)
        └── presentation/   # bloc/, pages/, widgets/
```
Don't expose Drift rows or Dio DTOs directly to widgets — map at the data layer boundary. Keep feature code inside its feature folder; only put things in `core/` if genuinely shared. Don't add a layer, use-case class, or per-widget BLoC unless it earns its keep.

### Known pitfalls called out in the curriculum (§10)
- `DateTime(...)` has no `const` constructor — don't try to make a model with a `DateTime` field `const` without passing the timestamp in.
- `^` in Dart is bitwise XOR, not exponentiation — don't use it for backoff math (`1 << attempt` instead).
- `ProviderContainer` belongs to package `riverpod`, not package `provider` — don't mix them.
- When retrying a Dio request after refreshing auth, use `dio.fetch(requestOptions)` + `handler.resolve(response)` — `handler.next(requestOptions)` does not execute a request.
- Treat `connectivity_plus` as a hint only; a "connected" status doesn't guarantee a request will succeed.
- Firebase Dynamic Links shut down 2025-08-25 — don't use them for deep linking; use Android App Links / iOS Universal Links instead.
- Background sync scheduling (WorkManager / iOS background tasks) is best-effort — don't promise exact intervals.

### Testing priorities (test by risk, not by coverage %)
Highest priority: database migrations, note create/edit, draft preservation, search correctness, archive/delete, sync queue, conflict handling, auth refresh (if enabled), deep-link routing, notification scheduling/cancellation. Lower priority: framework behavior, trivial getters, static widget composition with no logic, third-party package internals. Test pyramid: many fast unit/database tests → focused widget tests for important states → a few stable end-to-end integration tests → manual device testing for permissions/notifications/accessibility/release builds.

## Commands

Standard Flutter tooling (no custom scripts/Makefile in this repo):

```bash
flutter pub get                 # install dependencies
flutter run                     # run the app (pick a connected device/emulator)
flutter analyze                 # static analysis (uses flutter_lints via analysis_options.yaml)
flutter test                    # run all tests in test/
flutter test test/widget_test.dart   # run a single test file
flutter test --plugin-name <name> --name "<test name>"  # run a single test by name
flutter build apk               # Android release build
flutter build ios               # iOS release build
```

Once code generation is introduced (Drift, `json_serializable` — see curriculum §4.1), the generation commands will be:
```bash
dart run build_runner build
dart run build_runner watch --delete-conflicting-outputs
```

## Environment

- Dart SDK: `^3.12.2` (Flutter 3.44.x stable channel)
- Android `applicationId` / `namespace`: `com.example.simply_notes` (placeholder — update before release, per curriculum §8.1)
- Linting: `flutter_lints` via `analysis_options.yaml`, defaults unmodified
