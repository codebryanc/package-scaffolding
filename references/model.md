#  Model

  - Models can be instantiated without any properties — all fields are optional (nullable).
  - Every property is prefixed with the first 3 letters of the model name.
    - Example: CredentialsModel → creName, crePassword, creIsEnabled
  - When a property references another model's identifier, keep the prefix of the source model, not the current one.
    - Example: a countryId field inside CredentialsModel stays couId, not creCouId.
  - All models include toJson and fromJson under a // [Mapping] region.
  - JSON keys match the property names exactly (camelCase).