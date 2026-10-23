# 06 — Code Style

## Dart / Flutter
- Follow `flutter_lints` + extra rules in `analysis_options.yaml`: `prefer_const_constructors`, `avoid_print`, `always_declare_return_types`, `require_trailing_commas`, `unawaited_futures`.
- Files `snake_case.dart`; classes `PascalCase`; variables/functions `camelCase`; constants `lowerCamelCase`; providers end with `Provider`.
- One public widget per file. Max ~250 lines per file; split before that.
- Widgets: prefer small `StatelessWidget`/`ConsumerWidget` classes over helper methods returning widgets. Use `const` wherever possible.
- No business logic, no Supabase calls, no `BuildContext` in domain/data layers.
- Immutable models with `freezed`; `copyWith` instead of mutation.
- Async: always handle loading/error via `AsyncValue`. Never `!` on nullable without a prior check; never swallow exceptions (`catch (_) {}`).
- Errors: repositories return `Result<T, AppFailure>`. Map Supabase/network errors to typed failures (`NetworkFailure`, `AuthFailure`, `PaymentFailure`, `NotFoundFailure`).
- Strings via l10n, colours/spacing/text styles via tokens/theme only.
- Imports: dart → package → project (relative inside a feature, package imports across features). Sorted.
- Comments explain *why*, not *what*. Public APIs get `///` docs.

## Example pattern
```dart
// domain
abstract interface class CourseRepository {
  Future<Result<List<Course>>> getPublishedCourses({String? categoryId});
}
// application
@riverpod
class CourseListNotifier extends _$CourseListNotifier { ... }
// presentation
class CourseListScreen extends ConsumerWidget { ... }
```

## TypeScript (Edge Functions)
- Strict mode, no `any`. Validate input with zod. One function = one folder with `index.ts` + `index.test.ts`.
- Shared code in `supabase/functions/_shared/`. Return `{ error: { code, message } }` JSON with correct HTTP status.
- Never log secrets or full request bodies.

## SQL
- `snake_case`, plural table names, migration files `YYYYMMDDHHMM_description.sql`, each migration reversible or documented.

## Git
- Branch `feat/<phase>-<short-name>`; Conventional Commits (`feat:`, `fix:`, `test:`, `chore:`, `docs:`).
- Small commits; do not mix refactors with features.

## Dependencies
Prefer first-party/popular maintained packages. Before adding one: state reason, last release date, license. Pin versions.
