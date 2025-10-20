# GitHub Copilot Instructions for AIndroid

## Project Overview

AIndroid is an Android application that provides AI-powered file organization capabilities. The app allows users to intelligently organize files from a SOURCE directory into a structured LIBRARY using AI to categorize and sort content.

### Key Features
- Dual-root file organization (SOURCE → LIBRARY)
- Two-level hierarchy (Category/Subcategory or Category/_root)
- AI-driven content categorization
- Preview plans before applying changes
- Full undo support with move logging
- Smart name clash resolution
- Storage Access Framework (SAF) integration for Android file system access

## Technology Stack

### Core Technologies
- **Language**: Kotlin 1.9.20
- **Build System**: Gradle 8.2 with Kotlin DSL
- **Android Gradle Plugin**: 8.1.4
- **UI Framework**: Jetpack Compose with Material 3
- **Minimum SDK**: API 24 (Android 7.0 Nougat)
- **Target SDK**: API 34 (Android 14)

### Key Dependencies
- **androidx.compose**: Modern declarative UI toolkit
- **androidx.documentfile**: Storage Access Framework support
- **kotlinx-serialization-json**: JSON serialization/deserialization
- **okhttp**: HTTP client for backend communication
- **androidx.lifecycle**: Lifecycle-aware components

## Architecture

### File Structure
```
app/src/main/java/com/example/aindroid/
├── MainActivity.kt           # Main entry point
└── ui/theme/                 # Compose theme definitions
    ├── Color.kt
    ├── Theme.kt
    └── Type.kt
```

**Note**: This is the current structure. As the project grows, expect additional directories for:
- `data/`: Data models, repositories, and data sources
- `domain/`: Business logic and use cases
- `ui/`: Screen composables and ViewModels
- `util/`: Utility functions and helpers

### Design Patterns
- **MVVM**: Use ViewModel for business logic, Composables for UI
- **Single Activity**: All screens as Composable functions
- **Repository Pattern**: Separate data access from business logic
- **Dependency Injection**: Prefer constructor injection for testability

## Coding Standards

### Kotlin Style
- Follow official [Kotlin coding conventions](https://kotlinlang.org/docs/coding-conventions.html)
- Use meaningful variable and function names
- Prefer `val` over `var` for immutability
- Use data classes for model objects
- Leverage Kotlin's null safety features

### Compose Best Practices
- Keep composables small and focused
- Use `remember` and `rememberSaveable` appropriately
- Hoist state to the appropriate level
- Use `Modifier` as the first optional parameter
- Provide preview functions for all UI components
- Use Material 3 components consistently

### Code Organization
- Group related functions together
- Keep files under 500 lines when possible
- Use meaningful package names
- Separate UI, data, and domain layers

## Project-Specific Guidelines

### File Organization System
- **SOURCE**: The unsorted root directory to organize
- **LIBRARY**: The organized destination with two-level structure
- **Categories**: Top-level folders in LIBRARY (e.g., "Media", "Documents")
- **Subcategories**: Second-level folders or "_root" for category-level files
- **Workspace**: Cache directory at `cache/organizer/<hash>/`

### Data Models
When working with file organization features:
- `SourceOverview`: Contains SOURCE inventory with top 5 biggest files per folder
- `LibraryIndex`: Two-level map of existing categories/subcategories
- `Plan`: List of placements and optional new folders to create
- `MoveOp`: Records actual move operations for undo functionality

### SAF Integration
- Always use `DocumentFile` and `ContentResolver` for file operations
- Prefer `DocumentsContract.moveDocument` when available
- Fall back to copy+delete when move is not supported
- Implement `uniqueChildName()` for name clash resolution (e.g., "file (2).ext")
- Persist URI permissions with `takePersistableUriPermission()`

### JSON Serialization
- Use `kotlinx.serialization.json` for all JSON operations
- Define `@Serializable` data classes for all JSON structures
- Handle optional fields with nullable types or default values
- Follow the schemas defined in `AI_File_Organizer_Plan.md`

### Backend Communication
- Keep API keys server-side only
- Use a proxy service for AI planning requests
- Endpoint: `POST /plan` with SourceOverview and LibraryIndex
- Handle network errors gracefully with user feedback

## Building and Testing

### Build Commands
```bash
# Build the project
./gradlew build

# Install debug build
./gradlew installDebug

# Clean build artifacts
./gradlew clean

# Check tasks
./gradlew tasks
```

### Testing Guidelines
- Write unit tests for business logic
- Use Compose testing for UI components
- Test SAF operations with mock DocumentFile
- Verify JSON serialization/deserialization
- Test name clash resolution edge cases
- Validate undo operations

## Common Patterns

### Composable Functions
```kotlin
@Composable
fun MyComponent(
    modifier: Modifier = Modifier,
    viewModel: MyViewModel = viewModel()
) {
    // Component implementation
}

@Preview(showBackground = true)
@Composable
fun MyComponentPreview() {
    AIndroidTheme {
        MyComponent()
    }
}
```

### State Management
```kotlin
@Composable
fun StatefulComponent() {
    var state by remember { mutableStateOf(initialValue) }
    
    // Use state in UI
}
```

### JSON Serialization
```kotlin
@Serializable
data class MyData(
    val field1: String,
    val field2: Int,
    @SerialName("custom_name")
    val field3: String? = null
)

// Encode
val json = Json.encodeToString(myData)

// Decode
val data = Json.decodeFromString<MyData>(jsonString)
```

## Error Handling

### General Principles
- Use `Result<T>` for operations that can fail
- Provide meaningful error messages
- Log errors appropriately
- Show user-friendly error dialogs in UI
- Handle SAF permissions errors gracefully

### Common Error Scenarios
- Missing SAF permissions: Request again or show guidance
- Source item not found: Skip with logged reason
- Destination already exists: Apply name clash resolution
- Network errors: Show retry option
- Undo failures: Log and continue with other operations

## Performance Considerations

### File Operations
- Use coroutines for file scanning and moving
- Show progress UI for long operations
- Support cancellation of operations
- Batch operations when possible
- Use background threads for I/O operations

### Compose Performance
- Avoid unnecessary recompositions
- Use `derivedStateOf` for computed values
- Implement proper `equals()` for state objects
- Profile with Layout Inspector

## Security and Privacy

### API Keys
- **NEVER** store API keys in the Android app
- Use server-side proxy for AI services
- Keep credentials in backend environment variables

### Permissions
- Request only necessary permissions
- Explain permission requirements to users
- Handle permission denials gracefully
- Use SAF for scoped storage access

## Documentation

### Code Comments
- Add KDoc for public APIs
- Explain complex algorithms
- Document non-obvious business rules
- Keep comments up to date with code changes

### README and Docs
- Update `PROJECT_SETUP.md` for setup changes
- Keep `AI_File_Organizer_Plan.md` synchronized with implementation
- Document breaking changes
- Provide usage examples

## Git Workflow

### Commit Messages
- Use clear, descriptive commit messages
- Start with a verb in present tense
- Reference issue numbers when applicable
- Keep commits focused and atomic

### Code Reviews
- Keep changes focused and reviewable
- Write tests for new features
- Update documentation
- Ensure builds pass before review

## Additional Resources

- [Kotlin Documentation](https://kotlinlang.org/docs/home.html)
- [Jetpack Compose Documentation](https://developer.android.com/jetpack/compose)
- [Android Developer Guides](https://developer.android.com/guide)
- [Storage Access Framework](https://developer.android.com/guide/topics/providers/document-provider)
- [Kotlinx Serialization](https://github.com/Kotlin/kotlinx.serialization)
