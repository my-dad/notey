# Repository Guidelines

## Project Structure & Module Organization
This repository is a Flutter learning project for a local-first note app. Current runtime code is in `lib/`, tests are in `test/`, and platform runners are in `android/` and `ios/`. Root planning and constraints live in `Flutter_Learning_Curriculum.md` and `CLAUDE.md`.

Build toward the curriculum’s feature-first layout: `lib/app/` for app shell and routing, `lib/core/` for truly shared infrastructure, and `lib/features/<feature>/data|domain|presentation/` for product code. Keep DTOs, Drift rows, and UI models separated by mappers rather than leaking data-layer types into widgets.

## Build, Test, and Development Commands
- `flutter pub get` installs Dart and Flutter dependencies.
- `flutter run` launches the app on a connected emulator or device.
- `flutter analyze` runs static analysis with `flutter_lints`.
- `flutter test` runs all tests in `test/`.
- `flutter test test/widget_test.dart` runs a single widget test file.
- `flutter build apk` builds an Android release artifact.
- `flutter build ios` validates the iOS build when platform-sensitive work lands.
- `dart run build_runner build` generates code once Drift or `json_serializable` is added.
- `dart run build_runner watch --delete-conflicting-outputs` regenerates code during active schema/model work.

## Coding Style & Naming Conventions
Use 2-space indentation and run `dart format .` before submitting changes. Follow Dart naming: `UpperCamelCase` for types, `lowerCamelCase` for members, and `snake_case.dart` for files. Prefer immutable state, small widgets, and clear composition. Use Cubit by default, BLoC only where event concurrency or debounce logic adds value. Keep ephemeral UI state such as controllers, focus, and selection local to widgets.

## Testing Guidelines
Use `test`, `flutter_test`, `bloc_test`, and later `integration_test` as the curriculum introduces them. Name files `*_test.dart` and place them under `test/`. Test by risk, not by coverage percentage: prioritize note CRUD, draft preservation, search, archive/delete, database migrations, sync queue behavior, deep-link routing, and notification scheduling. Run `flutter analyze` and `flutter test` before a PR, and smoke-test iOS whenever changes touch plugins, storage, routing, notifications, or builds.

## Commit & Pull Request Guidelines
Recent history uses short, imperative commit subjects such as `Add README.md and vs_code.json for extension discovery cache`. Follow that pattern: one clear action per commit line. Pull requests should include a concise summary, testing notes, linked issue or task context, and screenshots or screen recordings for visible UI changes.

## Configuration Notes
The package is private (`publish_to: 'none'`). Keep secrets, keystores, and generated credentials out of git. Follow the curriculum’s technical choices unless there is a strong reason not to: `go_router` for navigation, `RepositoryProvider` and `BlocProvider` for composition, Drift for local persistence, Dio for networking, and `flutter_secure_storage` for tokens or secrets.
