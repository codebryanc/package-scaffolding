# Error Handling

## Pattern: Either (Failure Left, Success Right)

Use the custom `Either` sealed class to represent operations that can fail:

- **Left** → failure (`Failure`)
- **Right** → success (`T`)

```dart
sealed class Either<L, R> {
  const Either();

  T fold<T>(T Function(L) onLeft, T Function(R) onRight) => switch (this) {
    Left<L, R>(:final value) => onLeft(value),
    Right<L, R>(:final value) => onRight(value),
  };
}

final class Left<L, R> extends Either<L, R> {
  const Left(this.value);
  final L value;
}

final class Right<L, R> extends Either<L, R> {
  const Right(this.value);
  final R value;
}

typedef ResultEither<T> = Either<Failure, T>;
```

## Shared Model Location

`Shared` → `lib/src/common/model/common/result_model.dart`

> **Rule — never create an error model inside the feature package.**
> Do NOT scaffold `lib/common/models/errors/error_model.dart` or any class named `ErrorModel`.
> Errors are represented by `Failure` (Left branch) and `ResultModel` (Right branch) from the `Shared` package.
> Import `ResultModel` from Shared; do not redefine its fields.

## ResultModel Properties

```dart
class ResultModel {
  // [Properties]
  String? guidId;
  int? id;
  late bool result;
  dynamic data;
  String? message;

  // [Error Properties]
  String? uiMessage;
  String? applicationMessage;
  String? domainMessage;
  String? infrastructureMessage;
  String? databaseMessage;

  // [Network Properties]
  int? statusCode;
}
```

### Property Descriptions

| Property | Layer | Purpose |
|---|---|---|
| `guidId` | General | Unique identifier (UUID) |
| `id` | General | Numeric identifier |
| `result` | General | `true` = success, `false` = failure |
| `data` | General | Payload from the operation |
| `message` | General | Generic message |
| `uiMessage` | Presentation | User-facing error message |
| `applicationMessage` | Application | App-layer error detail |
| `domainMessage` | Domain | Business rule violation message |
| `infrastructureMessage` | Infrastructure | Infrastructure-layer error |
| `databaseMessage` | Infrastructure | Database-specific error |
| `statusCode` | Network | HTTP status code |

## Usage Example

```dart
// Repository returns Either
Future<Either<Failure, ResultModel>> fetchSomething();

// Usage in use case / datasource
final response = await repository.fetchSomething();
return response.fold(
  (failure) => Left(failure),
  (model) => Right(model),
);
```

## Notes

- Always map API/network errors to the appropriate message field by layer.
- `result == false` with a non-null error field signals a domain/app-level failure even with HTTP 200.
- `statusCode` is used to distinguish network-level failures (4xx, 5xx).

---

## Handling `await` with `.fold` in BLoC

When a use case or application method returns `Future<Either<Failure, T>>`, the BLoC awaits the result and uses `.fold` to branch into two paths: the **left** (failure) and the **right** (success).

### `.fold` anatomy

```dart
final result = await _application.doSomething(params);

result.fold(
  (failure) {
    // Left → Failure
    // Called when the operation failed.
    // `failure` is a Failure object — use failure.message for the error.
    emit(FeatureErrorState(message: failure.message));
  },
  (model) {
    // Right → Success
    // Called when the operation succeeded.
    // `model` is the expected return type T (e.g. ResultModel, List<X>, etc.)
    emit(FeatureSuccessState(data: model));
  },
);
```

| Branch | Receives | When it runs | Emit |
|---|---|---|---|
| Left `(failure)` | `Failure` | Operation failed (network, domain, validation) | Error state with `failure.message` |
| Right `(model)` | `T` (your return type) | Operation succeeded | Success state with the returned data |

### Full BLoC example

```dart
on<FetchFeatureEvent>(_onFetchFeature);

Future<void> _onFetchFeature(
  FetchFeatureEvent event,
  Emitter<FeatureState> emit,
) async {
  emit(const FeatureLoadingState());

  final result = await _application.fetchFeature(event.id);

  result.fold(
    (failure) => emit(FeatureErrorState(message: failure.message)),
    (model) => emit(FeatureLoadedState(feature: model)),
  );
}
```

### States that map to each branch

```dart
// Loading — emitted before the await
class FeatureLoadingState extends FeatureState { const FeatureLoadingState(); }

// Success — emitted in the Right branch
class FeatureLoadedState extends FeatureState {
  const FeatureLoadedState({required this.feature});
  final FeatureModel feature;
}

// Error — emitted in the Left branch
class FeatureErrorState extends FeatureState {
  const FeatureErrorState({required this.message});
  final String message;
}
```

### Rules

- Always emit a loading state **before** the `await`.
- Never access `failure` inside the right branch or `model` inside the left branch — they are scoped to their own side.
- Use `failure.message` as the user-facing message unless a specific `uiMessage` is available on the `ResultModel`.
- For operations that return `ResultModel`, also check `model.result == false` inside the right branch if the API can signal a logical failure with HTTP 200.
