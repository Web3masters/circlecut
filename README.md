# 🛠️ CircleCut Guide

This guide provides detailed instructions for setting up and testing the CircleCut Android application.

## 📋 Prerequisites

### Required Software
- **Android Studio**: Arctic Fox (2020.3.1) or later
- **Java Development Kit**: JDK 11 or later
- **Android SDK**: API level 21 (Android 5.0) minimum
- **Git**: For version control
- **Node.js**: For additional tooling (optional)

### Required Accounts
- **Circle Developer Account**: For Programmable Wallet API access
- **Supabase Account**: For backend database services
- **GitHub Account**: For repository access

## 🔧 Environment Setup

### 1. Android Studio Configuration

#### SDK Setup
```bash
# Install required SDK platforms
Android SDK Platform 33
Android SDK Platform 34
Android SDK Build-Tools 33.0.0+
```

#### Gradle Configuration
Ensure your `gradle.properties` includes:
```properties
android.useAndroidX=true
android.enableJetifier=true
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
```

### 2. Circle SDK Integration

#### Local AAR Setup
The project uses a local Circle SDK AAR file:
```
libs/circle-sdk-1.0.1189.aar
libs/circle-sdk-1.0.1189.pom
```

#### API Configuration
Update `ApiManager.kt` with your Circle API credentials:
```kotlin
class ApiManager {
    private var apiKey = "YOUR_CIRCLE_API_KEY_HERE"
    
    // Circle API endpoints
    private val baseUrl = "https://api.circle.com/v1/w3s/"
}
```

#### Circle Developer Setup
1. Visit [Circle Developer Console](https://console.circle.com/)
2. Create a new project
3. Generate API keys
4. Configure webhook endpoints (optional)

### 3. Supabase Configuration

#### Database Setup
Create a new Supabase project and run these SQL commands:

```sql
-- Enable Row Level Security
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE expenses ENABLE ROW LEVEL SECURITY;

-- Users table
CREATE TABLE users (
    uid SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    number TEXT,
    password TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Expenses table
CREATE TABLE expenses (
    expenseid SERIAL PRIMARY KEY,
    uid1 INTEGER REFERENCES users(uid) NOT NULL,
    uid2 INTEGER REFERENCES users(uid) NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    currency TEXT DEFAULT 'USD',
    name TEXT NOT NULL,
    paidby INTEGER REFERENCES users(uid),
    "when" TEXT,
    issettled BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_expenses_uid1 ON expenses(uid1);
CREATE INDEX idx_expenses_uid2 ON expenses(uid2);
CREATE INDEX idx_expenses_settled ON expenses(issettled);
```

#### Application Configuration
Update `supabaseinit.kt`:
```kotlin
class supabaseinit {
    fun getsupa(url: String, key: String): SupabaseClient {
        val supabase = createSupabaseClient(
            supabaseUrl = "YOUR_SUPABASE_URL",
            supabaseKey = "YOUR_SUPABASE_ANON_KEY"
        ) {
            install(GoTrue)
            install(Postgrest)
        }
        return supabase
    }
}
```

## 🏗️ Project Structure

### Core Modules
```
app/src/main/java/com/example/circlecut/
├── ui/
│   ├── main/               # Main fragments and navigation
│   └── components/         # Reusable UI components
├── data/
│   ├── models/            # Data models (User, Expense, etc.)
│   ├── repositories/      # Data access layer
│   └── local/             # Local storage
├── network/
│   ├── api/               # API interfaces
│   └── interceptors/      # Network interceptors
├── utils/
│   ├── extensions/        # Kotlin extensions
│   └── helpers/           # Utility functions
└── di/                    # Dependency injection
```

### Key Files
- **MainActivity.kt**: App entry point
- **ApiManager.kt**: Circle API integration
- **SessionManager.kt**: User session management
- **supabaseinit.kt**: Database configuration
- **ExpenseAdapter.kt**: RecyclerView adapter for expenses

## 🔨 Build Configuration

### Gradle Dependencies
Key dependencies in `app/build.gradle`:
```kotlin
dependencies {
    // Circle SDK (local)
    implementation files('../libs/circle-sdk-1.0.1189.aar')
    
    // Supabase
    implementation platform("io.github.jan-tennert.supabase:bom:1.4.7")
    implementation 'io.github.jan-tennert.supabase:postgrest-kt'
    implementation 'io.github.jan-tennert.supabase:realtime-kt'
    
    // Networking
    implementation("io.ktor:ktor-client-android:2.3.6")
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.11.0'
    
    // UI Components
    implementation 'androidx.viewpager2:viewpager2:1.0.0'
    implementation 'com.google.android.material:material:1.9.0'
    
    // Coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
}
```

### Build Variants
Configure build variants for different environments:
```kotlin
android {
    buildTypes {
        debug {
            debuggable true
            applicationIdSuffix ".debug"
            buildConfigField "String", "API_BASE_URL", "\"https://api-sandbox.circle.com/v1/w3s/\""
        }
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            buildConfigField "String", "API_BASE_URL", "\"https://api.circle.com/v1/w3s/\""
        }
    }
}
```

## 🧪 Testing Setup

### Unit Tests
Create unit tests in `app/src/test/`:
```kotlin
@Test
fun testExpenseCalculation() {
    val expense = Expense("Dinner", "Split dinner bill", "J", 50.0, "USD")
    assertEquals(25.0, expense.cost?.div(2))
}
```

### Integration Tests
Create integration tests in `app/src/androidTest/`:
```kotlin
@Test
fun testDatabaseOperations() {
    // Test Supabase integration
}
```

### Running Tests
```bash
# Unit tests
./gradlew test

# Integration tests
./gradlew connectedAndroidTest

# All tests
./gradlew check
```

## 🚀 Development Workflow

### 1. Local Development
```bash
# Start development
git checkout -b feature/your-feature
./gradlew assembleDebug
./gradlew installDebug
```

### 2. Code Quality
```bash
# Lint check
./gradlew lint

# Format code
./gradlew ktlintFormat

# Static analysis
./gradlew detekt
```

### 3. Debugging

#### Circle SDK Debugging
Enable debug logging:
```kotlin
WalletSdk.init(
    applicationContext,
    WalletSdk.Configuration(
        "https://api.circle.com/v1/w3s/",
        "your-app-id",
        settingsManagement
    ).apply {
        isDebugMode = true
    }
)
```

#### Network Debugging
Add logging interceptor:
```kotlin
val logging = HttpLoggingInterceptor()
logging.setLevel(HttpLoggingInterceptor.Level.BODY)

val client = OkHttpClient.Builder()
    .addInterceptor(logging)
    .build()
```

## 🔍 Troubleshooting

### Common Issues

#### Circle SDK Build Errors
- Ensure AAR file is in `libs/` directory
- Check Gradle configuration
- Verify API key is set

#### Supabase Connection Issues
- Verify URL and API key
- Check network permissions
- Ensure RLS policies are configured

#### Compilation Errors
- Clean and rebuild: `./gradlew clean build`
- Invalidate caches in Android Studio
- Check Kotlin version compatibility

### Performance Optimization

#### Memory Management
- Use ViewBinding instead of findViewById
- Implement proper lifecycle management
- Avoid memory leaks in coroutines

#### Network Optimization
- Implement request caching
- Use connection pooling
- Handle offline scenarios

## 📱 Device Testing

### Emulator Setup
Recommended emulator configuration:
- **Device**: Pixel 5
- **API Level**: 30 (Android 11)
- **RAM**: 4GB
- **Storage**: 8GB

### Physical Device Testing
- Enable Developer Options
- Enable USB Debugging
- Install via ADB: `adb install app-debug.apk`

## 🔐 Security Considerations

### API Key Management
- Never commit API keys to version control
- Use environment variables or secure storage
- Rotate keys regularly

## 📊 Monitoring & Analytics

### Crash Reporting
Consider integrating:
- Firebase Crashlytics
- Sentry
- Bugsnag

### Performance Monitoring
- Android Profiler
- Firebase Performance
- Custom metrics

---

Happy code testing! 🚀
