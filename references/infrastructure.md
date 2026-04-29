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

## RemoteDataSource

Define `_controller` and one private `String` per method as URL properties. Dio handles the base URL internally — never include it here. After each call, delegate to `mapping` to parse the response:

```dart
class FeatureRemoteDataSource {
  FeatureRemoteDataSource({required this.dio, required this.mapping});

  final Dio dio;
  final FeatureMapping mapping;

  // [Properties]
  final String _controller = '/feature';
  final String _save = '/save';
  final String _getById = '/get-by-id';

  // [Methods]
  Future<bool> save(FeatureRequestDto request) async {
    final response = await dio.post(
      '$_controller$_save',
      data: request.toJson(),
    );
    return mapping.toSaveResult(response.data);
  }

  Future<FeatureModel> getById(String id) async {
    final response = await dio.get('$_controller$_getById/$id');
    return mapping.toModel(response.data);
  }
}
```

Rules:
- `_controller` is the resource path (e.g. `/feature`)
- Each method has its own private path property (e.g. `_save`, `_getById`)
- Full URL = `'$_controller$_methodPath'` — Dio's `baseUrl` provides the host
- Never inline a full URL string inside a method body

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
