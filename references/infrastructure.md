# Infrastructure Layer Reference

## Responsibilities

- Implements the repository interface defined in domain
- Contains DTOs, mapping, local and remote data sources
- DTOs are internal — never export them outside of infrastructure

## DTOs

Only used internally to transfer data between layers inside infrastructure:

```dart
// request
class FeatureRequestDto {
  const FeatureRequestDto({required this.name});
  final String name;
}

// response (only if needed)
class FeatureResponseDto {
  const FeatureResponseDto({required this.name});
  final String name;
}
```

## RepositoryImpl

Receives both data sources and decides which one to call via `_getConnection()`.
Always use explicit `if/else` with a `result` variable:

```dart
class FeatureRepositoryImpl implements FeatureRepository {
  FeatureRepositoryImpl({
    required this.localDataSource,
    required this.remoteDataSource,
    required this.mapping,
  });

  final FeatureLocalDataSource localDataSource;
  final FeatureRemoteDataSource remoteDataSource;
  final FeatureMapping mapping;

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

  bool _getConnection() => false;
}
```
