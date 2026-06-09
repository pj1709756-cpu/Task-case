# TaskCash — Complete Setup Guide

## 📁 Project Structure

```
taskcash/
├── lib/
│   ├── main.dart
│   ├── firebase_options.dart
│   ├── core/
│   │   ├── constants/app_constants.dart
│   │   ├── theme/app_theme.dart
│   │   └── widgets/
│   │       ├── custom_button.dart
│   │       ├── custom_text_field.dart
│   │       ├── gradient_background.dart
│   │       └── shimmer_widget.dart
│   ├── data/
│   │   ├── models/
│   │   │   ├── user_model.dart
│   │   │   ├── task_model.dart
│   │   │   └── transaction_model.dart
│   │   └── services/
│   │       ├── auth_service.dart
│   │       ├── task_service.dart
│   │       └── withdrawal_service.dart
│   └── presentation/
│       ├── splash_screen.dart
│       ├── auth/
│       │   ├── auth_provider.dart
│       │   ├── login_screen.dart
│       │   ├── signup_screen.dart
│       │   └── forgot_password_screen.dart
│       ├── home/
│       │   ├── main_screen.dart
│       │   └── home_screen.dart
│       ├── tasks/tasks_screen.dart
│       ├── wallet/wallet_screen.dart
│       ├── referral/referral_screen.dart
│       ├── profile/profile_screen.dart
│       └── admin/admin_panel_screen.dart
├── android/app/src/main/AndroidManifest.xml
├── firebase/
│   ├── firestore.rules
│   └── storage.rules
└── pubspec.yaml
```

---

## 🔥 Step 1 — Create Firebase Project

1. Go to https://console.firebase.google.com
2. Click **Add project** → name it `TaskCash`
3. Disable Google Analytics (optional) → **Create project**

---

## 📱 Step 2 — Add Android App to Firebase

1. In Firebase Console → **Project Settings** → **Add app** → Android icon
2. Android package name: `com.taskcash.app`
3. App nickname: `TaskCash`
4. Download **google-services.json**
5. Place it at: `android/app/google-services.json`

---

## ⚙️ Step 3 — Configure android/build.gradle files

**android/build.gradle** (project-level):
```gradle
buildscript {
    dependencies {
        classpath 'com.google.gms:google-services:4.4.1'
    }
}
```

**android/app/build.gradle** (app-level):
```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'com.google.gms.google-services'  // ← add this
}

android {
    compileSdkVersion 34
    defaultConfig {
        applicationId "com.taskcash.app"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode 1
        versionName "1.0.0"
        multiDexEnabled true
    }
}

dependencies {
    implementation platform('com.google.firebase:firebase-bom:33.1.0')
    implementation 'com.google.firebase:firebase-analytics'
}
```

---

## 🛠️ Step 4 — FlutterFire CLI Setup (Recommended)

```bash
# Install FlutterFire CLI
dart pub global activate flutterfire_cli

# In your project root
flutterfire configure --project=YOUR_FIREBASE_PROJECT_ID
```

This auto-generates `lib/firebase_options.dart` with correct values.

> If you skip CLI, manually fill `lib/firebase_options.dart` with values from `google-services.json`.

---

## 🔐 Step 5 — Enable Firebase Services

### Authentication
1. Firebase Console → **Authentication** → **Get started**
2. Enable **Email/Password**
3. Enable **Google** (add your SHA-1 fingerprint)

Get SHA-1:
```bash
cd android
./gradlew signingReport
# Copy the SHA1 from "debug" variant
```
Add SHA-1 in Firebase Console → Project Settings → Your Android App → Add fingerprint

### Firestore Database
1. Firebase Console → **Firestore Database** → **Create database**
2. Start in **production mode**
3. Choose region (e.g., `asia-south1` for India)
4. After creation, go to **Rules** tab → paste contents of `firebase/firestore.rules` → **Publish**

### Firebase Storage
1. Firebase Console → **Storage** → **Get started**
2. Start in **production mode**
3. Go to **Rules** tab → paste contents of `firebase/storage.rules` → **Publish**

### Cloud Messaging (FCM)
1. Firebase Console → **Cloud Messaging** → already enabled
2. For push notifications, add your **Server Key** to backend if needed

---

## 📦 Step 6 — Install Dependencies

```bash
flutter pub get
```

---

## 🌱 Step 7 — Seed Initial Tasks in Firestore

Go to **Firestore Console** → **tasks** collection → Add documents:

### Daily Task 1
```json
{
  "title": "Complete Profile",
  "description": "Fill in all your profile details",
  "type": "daily",
  "coins": 50,
  "isActive": true,
  "completionCount": 0,
  "difficulty": 1,
  "createdAt": "<server timestamp>"
}
```

### Daily Task 2
```json
{
  "title": "Share TaskCash",
  "description": "Share the app on any social platform",
  "type": "daily",
  "coins": 75,
  "isActive": true,
  "completionCount": 0,
  "difficulty": 1,
  "createdAt": "<server timestamp>"
}
```

### Quiz Task
```json
{
  "title": "Daily Quiz",
  "description": "Answer today's question correctly",
  "type": "quiz",
  "coins": 100,
  "isActive": true,
  "completionCount": 0,
  "difficulty": 2,
  "extraData": {
    "question": "What is the capital of India?",
    "options": ["Mumbai", "New Delhi", "Kolkata", "Chennai"],
    "correctAnswer": 1
  },
  "createdAt": "<server timestamp>"
}
```

### Special Task
```json
{
  "title": "Rate the App",
  "description": "Rate TaskCash 5 stars on Play Store",
  "type": "special",
  "coins": 200,
  "isActive": true,
  "completionCount": 0,
  "difficulty": 1,
  "createdAt": "<server timestamp>"
}
```

---

## 👑 Step 8 — Set Up Admin Account

1. Run the app and **register** with your admin email
2. In Firestore Console → **users** collection → find your document
3. Set `isAdmin: true`
4. Copy your **UID** from the document ID
5. In `lib/core/constants/app_constants.dart`, replace:
   ```dart
   static const List<String> adminUIDs = ['REPLACE_WITH_ADMIN_UID'];
   ```
   with your actual UID.

---

## 🗄️ Firestore Collections Structure

```
users/
  {uid}/
    email: string
    name: string
    photoUrl: string | null
    totalCoins: number
    todayCoins: number
    totalEarnings: number
    referralCoins: number
    referralCode: string
    referredBy: string | null
    referralCount: number
    isAdmin: boolean
    isVerified: boolean
    isActive: boolean
    createdAt: timestamp
    lastSeen: timestamp
    lastCheckIn: timestamp | null
    streak: number
    level: number

tasks/
  {taskId}/
    title: string
    description: string
    type: "daily" | "quiz" | "watch_ad" | "referral" | "special"
    coins: number
    isActive: boolean
    completionCount: number
    difficulty: number (1-5)
    extraData: map | null   ← quiz questions go here
    createdAt: timestamp

user_tasks/
  {docId}/
    userId: string
    taskId: string
    isCompleted: boolean
    completedAt: timestamp
    coinsEarned: number

transactions/
  {txId}/
    userId: string
    type: "earned" | "withdrawn" | "referral_bonus" | "bonus" | "refund"
    coins: number
    description: string
    createdAt: timestamp
    taskId: string | null

withdrawals/
  {wId}/
    userId: string
    userName: string
    userEmail: string
    amount: number
    coins: number
    method: "upi" | "amazon_gift_card" | "google_play_gift_card"
    accountDetails: string
    status: "pending" | "approved" | "rejected"
    createdAt: timestamp
    processedAt: timestamp | null
    rejectionReason: string | null

announcements/
  {aId}/
    title: string
    body: string
    createdAt: timestamp
    isActive: boolean

settings/
  app/
    minWithdrawal: number
    coinsPerRupee: number
    maxAdsPerDay: number
    maintenanceMode: boolean
```

---

## 🏗️ Step 9 — Build APK

### Debug APK (for testing)
```bash
flutter build apk --debug
```
Output: `build/app/outputs/flutter-apk/app-debug.apk`

### Release APK
```bash
# 1. Generate keystore (one-time)
keytool -genkey -v -keystore ~/taskcash-release.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias taskcash

# 2. Create android/key.properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=taskcash
storeFile=/path/to/taskcash-release.jks

# 3. Update android/app/build.gradle
android {
    signingConfigs {
        release {
            def keystoreProperties = new Properties()
            keystoreProperties.load(new FileInputStream(rootProject.file('key.properties')))
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

# 4. Build
flutter build apk --release
```
Output: `build/app/outputs/flutter-apk/app-release.apk`

### App Bundle (for Play Store)
```bash
flutter build appbundle --release
```
Output: `build/app/outputs/bundle/release/app-release.aab`

---

## 💡 Coin Economy

| Action            | Coins  | ₹ Value |
|-------------------|--------|---------|
| Signup bonus      | 100    | ₹1.00   |
| Daily task        | 50     | ₹0.50   |
| Quiz correct      | 100    | ₹1.00   |
| Daily check-in    | 20+    | ₹0.20+  |
| Watch ad          | 30     | ₹0.30   |
| Refer a friend    | 500    | ₹5.00   |
| Join via referral | 200    | ₹2.00   |

**Conversion:** 100 coins = ₹1
**Min withdrawal:** ₹50 (5,000 coins)

---

## 🛡️ Security Checklist

- [x] Firestore rules restrict user data access
- [x] Storage rules limit file size and type
- [x] Admin actions gated by `isAdmin` flag in Firestore
- [x] Input validation on all forms
- [x] Duplicate task completion prevented server-side
- [x] Withdrawal balance check before deduction
- [x] Pending withdrawal check (one at a time)
- [ ] Add Firebase App Check for production
- [ ] Add server-side Cloud Functions for critical coin operations

---

## 🚀 Production Checklist

- [ ] Replace `firebase_options.dart` with real values
- [ ] Set real admin UID in `app_constants.dart`
- [ ] Generate release keystore
- [ ] Enable Firebase App Check
- [ ] Add real AdMob ads (replace simulated watch-ad)
- [ ] Set up Cloud Functions for server-side coin validation
- [ ] Configure FCM for push notifications
- [ ] Add crash reporting (Firebase Crashlytics)
- [ ] Set up Firebase Remote Config for dynamic coin values
- [ ] Privacy Policy & Terms of Service pages
- [ ] Test on multiple Android versions (API 21+)

---

## 📞 Support

For issues or customization requests, check:
- Flutter docs: https://docs.flutter.dev
- Firebase docs: https://firebase.google.com/docs
- FlutterFire: https://firebase.flutter.dev
