# Tier Clean Architecture (opt-in) — flutter-architecture

Solo leer/aplicar esto si el usuario confirmó que el proyecto es grande y necesita esta capa extra (`arquitectura: clean` en Args, o lo pide explícito). Por default el patrón usa el tier simple de `references/scaffold.md` — no mezclar ambos sin que el usuario lo haya pedido.

## Qué cambia vs el tier simple

El tier simple tiene `data/` y `presentation/` por feature, el repository se consume directo desde el provider. Clean Architecture agrega una capa `domain/` en el medio, con **inversión de dependencia real**: `presentation/` y `data/` dependen de `domain/`, nunca al revés.

Conserva convenciones globales del tier simple: `snake_case`, tests espejo bajo `test/`, `integration_test/` solo para journeys críticos y ningún import directo entre features. `domain/` es límite interno del feature, no un nuevo nivel global.

```
lib/features/<feature>/
├── data/
│   ├── models/
│   │   └── <model>_dto.dart          # DTO de la API (freezed + json_serializable)
│   └── repositories/
│       └── <feature>_repository_impl.dart   # implementa la interfaz de domain/
├── domain/
│   ├── entities/
│   │   └── <entity>.dart             # entidad de negocio, SIN json ni dependencias de data
│   ├── repositories/
│   │   └── <feature>_repository.dart # abstract class, el CONTRATO (interfaz)
│   └── usecases/
│       └── get_<feature>_list.dart   # una clase por caso de uso, un metodo call()
└── presentation/
    ├── providers/
    │   └── <feature>_notifier.dart   # llama al usecase, NUNCA al repository directo
    └── screens/
```

## Reglas de esta capa

1. **`domain/` no importa nada de `data/` ni `presentation/`.** Es la capa más interna, no depende de Flutter ni de dio ni de freezed/json — entidades planas de Dart puro.
2. **El contrato vive en `domain/repositories/`** como `abstract class`, con métodos que devuelven `Either<Failure, Entity>`. `data/repositories/<feature>_repository_impl.dart` lo implementa, traduciendo DTOs de la API a entidades de dominio.
3. **Un usecase por operación**, clase con un solo método `call()` (permite invocarla como función: `await getPostsList()`). El usecase orquesta 1+ repository, nunca al revés (el repository no conoce usecases).
4. **`presentation/` solo conoce `domain/`** — el provider (`@riverpod` AsyncNotifier) inyecta el usecase, nunca el repository ni el DTO directo.
5. **DI:** con Riverpod, el "wiring" (`repositoryProvider` devolviendo la implementación concreta detrás del tipo abstracto) reemplaza a un contenedor de DI tipo `get_it` para la mayoría de casos — usar `get_it` solo si hace falta resolver algo fuera del árbol de widgets/providers (ej. en un background isolate).

## Ejemplo mínimo

```dart
// domain/repositories/posts_repository.dart
abstract class PostsRepository {
  Future<Either<Failure, List<Post>>> getPosts();
}

// domain/usecases/get_posts_list.dart
class GetPostsList {
  GetPostsList(this._repository);
  final PostsRepository _repository;

  Future<Either<Failure, List<Post>>> call() => _repository.getPosts();
}

// data/repositories/posts_repository_impl.dart
class PostsRepositoryImpl implements PostsRepository {
  PostsRepositoryImpl(this._dio);
  final Dio _dio;

  @override
  Future<Either<Failure, List<Post>>> getPosts() async {
    // fetch + map PostDto -> Post (entidad de dominio)
  }
}

// presentation/providers/posts_notifier.dart
@riverpod
class PostsNotifier extends _$PostsNotifier {
  @override
  Future<List<Post>> build() async {
    final usecase = ref.watch(getPostsListProvider);
    final result = await usecase();
    return result.fold((f) => throw f, (posts) => posts);
  }
}
```

## Cuándo NO usar este tier

Si el proyecto es chico/mediano, esta capa agrega archivos y saltos sin beneficio real — el tier simple (repository directo desde el provider) ya separa negocio de UI lo suficiente. Escalar a Clean solo cuando: el equipo es grande (varios devs tocando el mismo feature en paralelo), hay múltiples fuentes de datos por feature (ej. API + cache local + fallback), o se necesita testear reglas de negocio 100% aisladas de Flutter/dio.
