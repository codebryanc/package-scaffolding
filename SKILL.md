---
name: package-scaffolding
description: Scaffold a Flutter feature following the architecture. Use when the user asks to create a feature, scaffold a module, or implement a new flow.
references:
  - references/ui.md
  - references/domain.md
  - references/application.md
  - references/infrastructure.md
  - references/common.md
  - references/errors.md
  - references/database.md
  - references/model.md
---

Generate all files for a Flutter feature following these strict rules:

## Architecture Rules

- NO Use Cases
- NO Equatable, NO `@override List<Object?> get props`
- All code and comments in English
- DTOs are internal to Infrastructure — never export them
- NO try-catch inside BLoC event handlers — error handling belongs in Application layer

## Layers & Responsibilities

**Common** → Global models shared across all layers, network 
**Domain** → Pure validation rules only (no models)  
**Application** → Orchestrates the step-by-step flow (validate → build model → call repository)  
**Infrastructure** → Repository, RemoteDataSource, DTOs (maps Common model ↔ JSON)  
**UI** → BLoC (events, states, bloc file) + Page  + Widgets (if needed)

## BLoC States

Only three states — Initial, Loading, Success. Nothing else.

```dart
abstract class FeatureState { const FeatureState(); }
class FeatureInitial extends FeatureState { const FeatureInitial(); }
class FeatureLoading extends FeatureState { const FeatureLoading(); }
class FeatureSuccess extends FeatureState {
  const FeatureSuccess();
}
```

## Result Pattern

### Plain types (bool, model, etc.)

Always define a `result` variable at the start, assign it, and return it at the end. Never use early returns or ternary for this:

```dart
// ✅ correct
bool result = false;
if (isConnected) {
  result = await remoteDataSource.save(request);
} else {
  result = await localDataSource.save(request);
}
return result;

// ❌ avoid
if (error != null) return false;
return repository.save();

// ❌ avoid
final bool result = isConnected ? await remote.save() : await local.save();
```

### Either<Exception, T>

When the return type is `Either<Exception, T>`, use multiple explicit `return` statements instead of a default `result` variable. This avoids the `const Left(...)` initialisation error and keeps the flow readable:

```dart
// ✅ correct
Future<Either<Exception, bool>> save({required String name}) async {
  final bool isConnected = _getConnection();

  if (isConnected) {
    return Right(await remoteDataSource.save(request));
  } else {
    return Right(await localDataSource.save(request));
  }
}

// ❌ avoid — triggers const-constructor init error
Either<Exception, bool> result = const Left(SomeException());
if (isConnected) {
  result = Right(await remoteDataSource.save(request));
}
return result;
```

## Repository Connectivity

`RepositoryImpl` always receives both `localDataSource` and `remoteDataSource`. It decides which one to call via a private `_getConnection()` method and an explicit `if/else`:

```dart
@override
Future<bool> save({required String name}) async {
  final request = mapping.toRequest(name);
  final bool isConnected = _getConnection();
  bool result = false;

  if (isConnected) {
    result = await remoteDataSource.save(request);
  } else {
    result = await localDataSource.save(request);
  }

  return result;
}

bool _getConnection() => false; // replace with real connectivity check
```

## File Structure to Generate

```
# Common

lib/common/di/dependency_injection.dart
lib/common/models/<feature>/<feature>_model.dart

# Feature

## Application

lib/features/<feature>/application/<feature>_application.dart

## Domain

lib/features/<feature>/domain/<feature>_domain.dart
lib/features/<feature>/domain/<feature>_repository.dart

## Infrastructure

lib/features/<feature>/infrastructure/dto/<feature>_request_dto.dart
lib/features/<feature>/infrastructure/dto/<feature>_response_dto.dart
lib/features/<feature>/infrastructure/mapping/<feature>_mapping.dart
lib/features/<feature>/infrastructure/data/<feature>_remote_data_source.dart
lib/features/<feature>/infrastructure/data/<feature>_local_data_source.dart
lib/features/<feature>/infrastructure/repositories/<feature>_repository_impl.dart

# UI

lib/features/<feature>/UI/bloc/<feature>_event.dart
lib/features/<feature>/UI/bloc/<feature>_state.dart
lib/features/<feature>/UI/bloc/<feature>_bloc.dart
lib/features/<feature>/UI/page/<feature>_page.dart
```

# Comments

- Please add comments regard region of code generated. On this way // [Properties] or // [Methods] or // [Mapping]

# Reference

- References file must be on relative path because this is a package