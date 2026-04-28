# Domain Layer Reference

## Validation Pattern

Always define a `result` variable, assign it, and return it at the end:

```dart
class FeatureDomain {
  const FeatureDomain();

  bool validateName(String? name) {
    bool result = name != null && name.trim().isNotEmpty;
    return result;
  }
}
```

## Repository

The repository interface lives in domain — it defines the contract without knowing anything about infrastructure:

```dart
abstract class FeatureRepository {
  Future<bool> save({required String name});
}
```

- Abstract class only — no implementation here
- Infrastructure implements it via `FeatureRepositoryImpl`
- Application depends on this interface, never on the implementation
