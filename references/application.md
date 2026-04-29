# Application Layer Reference

## Responsibilities

- Orchestrates the flow: validate → call repository
- Receives `domain` and `repository` by constructor — never instantiates dependencies internally
- Every method returns `Future<Either<Exception, T>>` where `T` is the actual data type (`bool`, a model, etc.)

## Pattern

Always validate with domain first. If validation fails, return `Left` immediately. For the happy path, define a single `result` variable and return it once at the end:

```dart
class FeatureApplication {
  const FeatureApplication({
    required this.domain,
    required this.repository,
  });

  final FeatureDomain domain;
  final FeatureRepository repository;

  Future<Either<Exception, bool>> save({required String name}) async {
    final bool nameIsValid = domain.validateName(name);

    if (!nameIsValid) {
      return Left(ValidationException(message: 'Invalid name'));
    }

    final bool result = await repository.save(name: name);
    return Right(result);
  }
}
```

Rules:
- Return type is always `Future<Either<Exception, T>>` — never a plain type
- Exception / error paths → multiple `return Left(...)` are allowed (early exits)
- App logic (happy path) → single `result` variable, one `return Right(result)` at the end
- Never mix: don't use a default `Either` variable for error paths
