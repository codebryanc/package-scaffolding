# Application Layer Reference

## Responsibilities

- Orchestrates the flow: validate → call repository
- Receives `domain` and `repository` by constructor — never instantiates dependencies internally
- Return type depends on the feature — can be `bool`, a model, or void

## Pattern

Always validate with domain first. If validation fails, repository is never called.
Define `result` at the start, assign it, return it at the end:

```dart
class FeatureApplication {
  const FeatureApplication({
    required this.domain,
    required this.repository,
  });

  final FeatureDomain domain;
  final FeatureRepository repository;

  Future<bool> save({required String name}) async {
    bool result = false;
    final bool nameIsValid = domain.validateName(name);

    if (nameIsValid) {
      result = await repository.save(name: name);
    }

    return result;
  }
}
```
