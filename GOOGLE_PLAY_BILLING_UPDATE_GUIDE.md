# Google Play Billing Library Upgrade Guide (v8.0.0+)

This guide provides step-by-step instructions to resolve the Google Play Console policy warning:
> **"App must use Google Play Billing Library version 8.0.0 or later"**

---

## 1. Why Did This Issue Occur?

- **Google Play Policy:** Google mandates that all apps and app updates must integrate **Google Play Billing Library 8.0.0 or later**.
- **Root Cause:** The project used `purchases_flutter: ^8.4.2` (RevenueCat SDK 8.x), which bundles Google Play Billing Library 6.x/7.x. Even if in-app purchases are only used on iOS, Flutter compiles the Android plugin into the Android App Bundle (`.aab`), causing Play Console to detect the outdated library.

---

## 2. Changes Already Applied in Code

The following changes have already been made to your project:

1. **`pubspec.yaml`**:
   - Incremented the app version:
     ```yaml
     version: 1.0.8+4
     ```
   - Upgraded RevenueCat's plugin to version 9.x (which uses Google Play Billing Library 8+):
     ```yaml
     purchases_flutter: ^9.16.1
     ```

2. **`android/app/build.gradle`**:
   - Added a Gradle resolution strategy to guarantee Google Play Billing Library resolves to version `8.0.0`:
     ```groovy
     configurations.all {
         resolutionStrategy {
             force 'com.android.billingclient:billing:8.0.0'
         }
     }
     ```

---

## 3. Step-by-Step Instructions to Follow

Follow these exact steps when you are ready to build and publish your app:

### Step 3.1: Fetch Updated Dependencies
In your project root terminal, run:
```bash
flutter clean
flutter pub get
```

### Step 3.2: Build the Release App Bundle
Generate the signed `.aab` for Google Play:
```bash
flutter build appbundle --release
```
The output bundle will be generated at:
```
build/app/outputs/bundle/release/app-release.aab
```

### Step 3.3: Upload to Google Play Console
1. Open the [Google Play Console](https://play.google.com/console).
2. Select your app (**Dhatri**).
3. Under **Test and release** in the left menu, prepare releases for:
   - **Production**
   - **Internal Testing** *(if active)*
   - **Closed Testing** *(if active)*
   - **Open Testing** *(if active)*

> ⚠️ **CRUCIAL - Update ALL Active Tracks:**  
> A common reason the policy warning does not disappear is having an older `.aab` active in a testing track (e.g., Internal or Closed testing). You must upload and roll out the new bundle to **every track that has an active release**.

4. Upload the newly generated `app-release.aab`.
5. Enter release notes, click **Save**, **Review release**, and **Start rollout**.

### Step 3.4: Verify Resolution
1. Wait a few minutes to hours for Google Play Console to analyze the new bundle.
2. Navigate to **Policy and programmes** > **Policy status**.
3. The warning:
   > *"App must use Google Play Billing Library version 8.0.0 or later"*  
   will transition to **Resolved** / **No issues found**.

---

## 4. Troubleshooting & Notes

- **If you get a build error during `flutter build appbundle` related to Kotlin version:**  
  Verify `android/settings.gradle` has Kotlin version set to `2.1.0` or later (currently set to `2.2.0`, which is fully compatible).
- **If you need to bump version code again for future releases:**  
  Update `version: 1.0.8+<number>` in `pubspec.yaml` (the number after `+` must always increase for Play Store uploads).
