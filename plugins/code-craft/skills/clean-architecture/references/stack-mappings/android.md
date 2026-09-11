# Android (Kotlin) stack mapping

> Answers the [mapping contract](../stack-detection.md#7-what-every-mapping-must-answer).
> Defaults here fill gaps only — the project's own conventions win (precedence §4).

## 1. Layer shapes

| Concept | Android shape |
|---|---|
| Entity / value object | `data class` / `value class` / sealed hierarchy in `domain/model/` — no Android or serialization annotations |
| Use case | class with `operator fun invoke(…)` (or the project's suffix — `*UseCase`, `*Interactor`), constructor-injected |
| Repository contract | `interface` in `data/repository/`, beside its impl, exposing domain types and `Flow` for observation |
| Repository implementation | class in `data/repository/` composing remote + local sources |
| Remote source | Retrofit/Ktor service + DTOs in `data/remote/` |
| Local source | Room `@Entity` + `@Dao` (or SQLDelight/DataStore) in `data/local/` |
| Mappers | extension functions `toDomain()` / `toEntity()` beside the models (or the project's mapper classes) |
| State holder | `ViewModel` exposing an immutable `StateFlow<UiState>`; sealed `UiState` with loading/content/empty/error. On Compose, the Route/Screen split, state ownership, and ViewModel lifetimes are `compose-screen-architecture` |

```kotlin
class SaveArticleUseCase @Inject constructor(
    private val repository: SavedArticlesRepository,
) {
    suspend operator fun invoke(id: ArticleId): AppResult<Unit> = repository.save(id)
}
// ViewModel calls the use case — never OrdersApi, the DAO, or the repository impl.
```

## 2. Role inventory hints (read Gradle dependency blocks)

| Role | Common packages |
|---|---|
| DI | `com.google.dagger:hilt-android` / `io.insert-koin` / plain Dagger / manual factories |
| HTTP | `com.squareup.retrofit2` / `io.ktor:ktor-client` / bare OkHttp |
| Storage | `androidx.room` / `app.cash.sqldelight` / `androidx.datastore` / Realm |
| Serialization | `kotlinx-serialization` / Moshi / Gson — annotations belong on DTOs only |
| UI | Compose vs XML Views — decides screen shape, not architecture |
| Navigation | `navigation-compose` / fragment NavController / Compose Destinations |
| Testing | JUnit 4/5, `mockk`/Mockito, `turbine` for Flow, Robolectric |

## 3. Banned imports per layer

`domain/` (and `usecase/`): no `android.*`, `androidx.*`, `retrofit2.*`, `okhttp3.*`,
`io.ktor.*`, `kotlinx.serialization.*`, Moshi/Gson, Room. `javax.inject`/`jakarta.inject`
annotations are the one tolerated framework touch — only if the project already does it.

Use cases may import **repository and gateway contracts** from `data/` — that is where those
interfaces live. What they must not import is anything concrete behind them: `*RepositoryImpl`,
data sources/clients, DTOs, or DB entities. Entities in `domain/model/` import nothing from
`data/` at all.

`presentation/`: no Retrofit/Room/DTO imports; state holders import use cases and domain
types, never `data.*`.

```bash
grep -rn "^import android\.\|^import androidx\.\|^import retrofit2\|^import okhttp3\|^import io.ktor\|^import kotlinx.serialization" app/src/main/java/**/features/*/domain/ && echo VIOLATION
grep -rn "\.data\." --include="*.kt" app/src/main/java/**/features/*/presentation/ | grep "^.*import" && echo VIOLATION
```

(Adapt paths to the project's package root; prefer the project's own architecture-test tooling
— Konsistic/ArchUnit — when present.)

## 4. Composition root

Hilt: `@Module`s where the project keeps them (`core/di/`, `features/<f>/data/di/`) installed
in components; `@HiltViewModel` for state holders. Koin: the module definitions +
`startKoin`. The Application class and DI modules are the only places that know concrete
implementations; `@Binds` ties the repository impl to its domain contract.

## 5. Test idiom

`./gradlew test` — JVM unit tests for domain, use cases (fake repositories), ViewModels (fake
use cases, `kotlinx-coroutines-test` + `turbine`), and mappers. Repository integration tests
use in-memory Room / MockWebServer per project habit. No emulator for business behavior.

## Greenfield fallbacks (no project convention found)

- Layout: `features/<feature>/{presentation,domain,data}` with `domain/{model,usecase}` and
  `data/{remote,local,repository,di}` — the repository interface and its impl both sit in
  `data/repository/`.
- Result/error idiom: a small sealed `AppResult`/`AppError` in `core/common/` (or Kotlin
  `Result` if the project already uses it at boundaries); technical exceptions are translated
  in `data/`, never rethrown inward.
- State holder: `ViewModel` + `StateFlow` + sealed `UiState` (loading/content/empty/error).
- Use-case naming: `VerbNounUseCase` with `operator fun invoke`.
