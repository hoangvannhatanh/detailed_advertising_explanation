# HƯỚNG DẪN TÍCH HỢP ADS - PHIÊN BẢN RÚT GỌN

> **Thư viện**: vtn_ads_libs v2.9.8  
> **Package**: com.github.devvtn:vtn_ads_libs:2.9.8

---

## 📌 TỔNG QUAN

### Thư viện sử dụng
```gradle
// Google Ads
implementation 'com.google.android.gms:play-services-ads:24.6.0'

// Facebook Mediation
implementation 'com.google.ads.mediation:facebook:6.20.0.0'
implementation 'com.facebook.android:facebook-android-sdk:18.1.3'

// VTN Ads Library
implementation 'com.github.devvtn:vtn_ads_libs:2.9.8'
implementation 'com.facebook.shimmer:shimmer:0.5.0'
```

### Các loại quảng cáo
- ✅ **App Open Ads** - Khi mở/resume app
- ✅ **Interstitial Ads** - Toàn màn hình
- ✅ **Native Ads** - Tích hợp vào UI
- ✅ **Banner Ads** - Băng rôn
- ✅ **Native Full Screen** - Native toàn màn hình

---

## 🚀 CÁC BƯỚC TÍCH HỢP

### BƯỚC 1: Cấu hình build.gradle (Root)

```gradle
allprojects {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}
```

### BƯỚC 2: Cấu hình AndroidManifest.xml

```xml
<manifest>
    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="com.google.android.gms.permission.AD_ID" />

    <application android:name=".MyApplication">
        
        <!-- Google AdMob App ID -->
        <meta-data
            android:name="com.google.android.gms.ads.APPLICATION_ID"
            android:value="@string/app_id" />

        <!-- Facebook App ID -->
        <meta-data
            android:name="com.facebook.sdk.ApplicationId"
            android:value="@string/facebook_app_id" />
        <meta-data
            android:name="com.facebook.sdk.ClientToken"
            android:value="@string/facebook_client_token" />
    </application>
</manifest>
```

### BƯỚC 3: Tạo file ads_id.xml

**File**: `app/src/main/res/values/ads_id.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Facebook -->
    <string name="facebook_app_id" translatable="false">YOUR_FB_APP_ID</string>
    <string name="facebook_client_token" translatable="false">YOUR_FB_CLIENT_TOKEN</string>
    <string name="adjust_token" translatable="false">YOUR_ADJUST_TOKEN</string>

    <!-- Google AdMob -->
    <string name="app_id" translatable="false">ca-app-pub-XXXXXXXX~XXXXXXXX</string>
    
    <!-- Test IDs (Development only) -->
    <!-- <string name="app_id">ca-app-pub-3940256099942544~3347511713</string> -->
    <!-- <string name="open_ads">ca-app-pub-3940256099942544/9257395921</string> -->
    <!-- <string name="inter_splash">ca-app-pub-3940256099942544/1033173712</string> -->
    <!-- <string name="native_all">ca-app-pub-3940256099942544/2247696110</string> -->
    <!-- <string name="banner_all">ca-app-pub-3940256099942544/9214589741</string> -->

    <!-- Real Ads IDs -->
    <string name="open_ads" translatable="false">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
    <string name="inter_splash" translatable="false">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
    <string name="native_home" translatable="false">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
    <string name="banner_all" translatable="false">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
    <!-- Thêm các ad units khác... -->
</resources>
```

### BƯỚC 4: Tạo Application Class

```kotlin
@HiltAndroidApp
class MyApplication : AdsApplication() {

    override fun onCreate() {
        super.onCreate()
        RemoteConfig.init(this)
        FirebaseApp.initializeApp(this)

        // Tắt App Open Ads cho một số màn hình
        AppOpenManager.getInstance().disableAppResumeWithActivity(SplashActivity::class.java)
    }

    override fun enableAdsResume() = true
    override fun getResumeAdId() = RemoteConfig.open_ads
    override fun buildDebug() = BuildConfig.DEBUG
    override fun enableAdjustTracking() = true
    override fun getAdjustToken() = getString(R.string.adjust_token)
    override fun getKeyRemoteIntervalShowInterstitial() = "interval_show_interstitial"
    override fun getListTestDeviceId() = null
    override fun getIntentOpenNotification() = Intent(this, SplashActivity::class.java)
}
```

### BƯỚC 5: Cấu hình Remote Config

**File**: `RemoteConfig.java`

```java
public class RemoteConfig {
    // Flags bật/tắt ads
    public static Boolean is_load_inter_splash = true;
    public static Boolean is_load_native_home = true;
    public static Boolean is_load_banner_all = true;
    public static long interval_show_interstitial = 15L;
    
    // Ad IDs
    public static String open_ads = "";
    public static String inter_splash = "";
    public static String native_home = "";
    public static String banner_all = "";

    public static void init(Context context) {
        open_ads = context.getString(R.string.open_ads);
        inter_splash = context.getString(R.string.inter_splash);
        native_home = context.getString(R.string.native_home);
        banner_all = context.getString(R.string.banner_all);
    }
}
```

### BƯỚC 6: Xin Consent (GDPR)

```kotlin
// Trong SplashActivity
private fun processAdConsent() {
    val consentHelper = ConsentHelper.getInstance(this)
    consentHelper.obtainConsentAndShow(this) {
        // Sau khi có consent, load ads
        loadAds()
    }
}
```

---

## 💡 SỬ DỤNG QUẢNG CÁO

### 1. APP OPEN ADS

Tự động hoạt động thông qua `AppOpenManager`. Chỉ cần config trong Application class.

```kotlin
// Tắt App Open cho màn hình cụ thể
AppOpenManager.getInstance().disableAppResumeWithActivity(SplashActivity::class.java)

// Bật lại trong Activity
AppOpenManager.getInstance().enableAppResumeWithActivity(MainActivity::class.java)
```

---

### 2. INTERSTITIAL ADS

```kotlin
// Load và show cùng lúc
private fun showInterstitial(nextAction: () -> Unit) {
    Admob.getInstance().loadAndShowInter(
        this,
        RemoteConfig.inter_splash,
        true, // show loading
        object : AdCallback() {
            override fun onNextAction() {
                nextAction.invoke()
            }
        }
    )
}

// Hoặc load và show với Native Full Screen (tăng fill rate)
private fun showInterWithNative(nextAction: () -> Unit) {
    Admob.getInstance().loadAndShowInterWithNativeFullScreen(
        this,
        RemoteConfig.inter_splash,
        RemoteConfig.native_full_splash,
        true,
        object : AdCallback() {
            override fun onNextAction() {
                nextAction.invoke()
            }
        }
    )
}
```

---

### 3. NATIVE ADS

**Layout XML:**
```xml
<FrameLayout
    android:id="@+id/native_ads_container"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

**Code:**
```kotlin
private fun loadNativeAds() {
    Admob.getInstance().loadNativeAd(
        this,
        RemoteConfig.native_home,
        object : NativeCallback() {
            override fun onNativeAdLoaded(nativeAd: NativeAd?) {
                val adView = LayoutInflater.from(this@YourActivity)
                    .inflate(R.layout.native_ads_layout, null) as NativeAdView
                binding.nativeAdsContainer.removeAllViews()
                binding.nativeAdsContainer.addView(adView)
                Admob.getInstance().pushAdsToViewCustom(nativeAd, adView)
            }

            override fun onAdFailedToLoad() {
                binding.nativeAdsContainer.visibility = View.GONE
            }
        }
    )
}
```

**Preload Pattern (Tối ưu):**
```kotlin
// Singleton để cache native ads
object AdsNativeConfig {
    var mNativeAdHome: NativeAd? = null
    
    fun loadNativeHome(context: Context) {
        if (mNativeAdHome == null) {
            Admob.getInstance().loadNativeAd(context, RemoteConfig.native_home, callback)
        }
    }
}

// Preload sớm trong Application
AdsNativeConfig.loadNativeHome(this)

// Show khi cần
AdsNativeConfig.showNativeHome(this, binding.nativeAdsContainer)
```

---

### 4. BANNER ADS

```kotlin
private fun loadBanner() {
    if (ConsentHelper.getInstance(this).canRequestAds() && 
        RemoteConfig.is_load_banner_all) {
        
        val config = BannerPlugin.Config()
        config.defaultAdUnitId = RemoteConfig.banner_all
        config.defaultBannerType = BannerPlugin.BannerType.Adaptive
        
        Admob.getInstance().loadBannerPlugin(
            this,
            binding.rlBanner,
            binding.include as ViewGroup,
            config
        )
        binding.loBanner.visibility = View.VISIBLE
    }
}
```

---

## ✅ BEST PRACTICES

### 1. Kiểm tra điều kiện trước khi show ads
```kotlin
private fun shouldShowAds(): Boolean {
    return RemoteConfig.is_load_inter_splash &&
           ConsentHelper.getInstance(this).canRequestAds() &&
           isNetworkAvailable(this) &&
           checkInterval()
}
```

### 2. Quản lý interval
```kotlin
private fun checkInterval(): Boolean {
    val lastTime = getSharedPreferences("app_prefs", MODE_PRIVATE)
        .getLong("last_show_ads", 0)
    val interval = RemoteConfig.interval_show_interstitial * 1000
    return (System.currentTimeMillis() - lastTime) >= interval
}

private fun saveLastShowTime() {
    getSharedPreferences("app_prefs", MODE_PRIVATE).edit()
        .putLong("last_show_ads", System.currentTimeMillis())
        .apply()
}
```

### 3. Preload ads
```kotlin
// Trong Application hoặc MainActivity
private fun preloadAds() {
    Handler(Looper.getMainLooper()).postDelayed({
        AdsNativeConfig.loadNativeHome(this)
        AdsInterConfig.loadInterTab(this)
    }, 2000)
}
```

### 4. Xử lý khi ads fail
```kotlin
Admob.getInstance().loadInterAds(this, adId, object : AdCallback() {
    override fun onAdFailedToLoad(error: LoadAdError?) {
        // Không block user, cho tiếp tục flow
        nextAction.invoke()
    }
})
```

### 5. Cleanup
```kotlin
override fun onDestroy() {
    super.onDestroy()
    mInterstitialAd = null
    mNativeAd?.destroy()
    mNativeAd = null
}
```

---

## 🔧 PROGUARD RULES

```proguard
# Google Ads
-keep class com.google.android.gms.ads.** { *; }
-keep interface com.google.android.gms.ads.** { *; }
-dontwarn com.google.android.gms.**

# Facebook Ads
-dontwarn com.facebook.infer.annotation.**

# Gson
-keep class com.google.gson.** { *; }
-keepattributes Signature
```

---

## ⚠️ LƯU Ý QUAN TRỌNG

| ⚠️ | Nội dung |
|----|----------|
| **Test Ads IDs** | Chỉ dùng khi development, KHÔNG dùng ở production |
| **Consent** | Luôn kiểm tra `ConsentHelper.getInstance(this).canRequestAds()` |
| **Interval** | Tối thiểu 15 giây giữa 2 lần show Interstitial |
| **Internet** | Kiểm tra kết nối trước khi load ads |
| **Cleanup** | Destroy ads trong `onDestroy()` |
| **Remote Config** | Dùng để bật/tắt ads từ xa mà không cần update app |

---

## 🎯 CHECKLIST TRƯỚC KHI RELEASE

- [ ] Đã thay Test Ads IDs bằng Real Ads IDs
- [ ] File `google-services.json` đã cấu hình đúng
- [ ] Package name khớp với AdMob console
- [ ] Meta-data `APPLICATION_ID` đã thêm vào Manifest
- [ ] Proguard rules đã thêm đầy đủ
- [ ] Test consent flow trên thiết bị thật
- [ ] Test ads hiển thị bình thường
- [ ] Remote Config đã setup trên Firebase

---

## 📊 CẤU TRÚC DỰ ÁN

```
app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/yourpackage/
│   │   │       ├── MyApplication.kt          # Application class
│   │   │       └── utils/
│   │   │           ├── RemoteConfig.java     # Remote config
│   │   │           ├── AdsNativeConfig.kt    # Native ads manager
│   │   │           └── AdsInterConfig.kt     # Inter ads manager
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   └── native_ads_layout.xml     # Native ads layout
│   │   │   └── values/
│   │   │       └── ads_id.xml                # Ads IDs
│   │   └── AndroidManifest.xml
│   ├── develop/
│   │   └── google-services.json              # Development config
│   └── production/
│       └── google-services.json              # Production config
├── build.gradle
└── proguard-rules.pro
```

---

## 📞 TÀI LIỆU THAM KHẢO

- **AdMob Console**: https://admob.google.com/
- **Firebase Console**: https://console.firebase.google.com/
- **VTN Ads Lib**: https://github.com/devvtn/vtn_ads_libs
- **Facebook Developer**: https://developers.facebook.com/

---

## 🚨 TROUBLESHOOTING NHANH

| Vấn đề | Giải pháp |
|--------|-----------|
| Ads không hiển thị | Kiểm tra consent, internet, Ad IDs |
| Crash khi show ads | Kiểm tra context null, Activity destroyed |
| Facebook mediation không work | Kiểm tra Facebook App ID, Client Token |
| Proguard error | Thêm keep rules cho ads classes |

---

**🎉 Hoàn tất! Bắt đầu kiếm tiền từ quảng cáo!**

