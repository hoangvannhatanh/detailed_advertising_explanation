# HƯỚNG DẪN TÍCH HỢP ADS - PHIÊN BẢN RÚT GỌN

> **Thư viện**: vtn_ads_libs v2.x.x
> **Package**: com.github.devvtn:vtn_ads_libs:2.x.x

---

## 📌 TỔNG QUAN

### Thư viện sử dụng
```gradle
// Google Ads
implementation 'com.google.android.gms:play-services-ads:x.x.x'

// Facebook Mediation
implementation 'com.google.ads.mediation:facebook:6.x.x.x'
implementation 'com.facebook.android:facebook-android-sdk:x.x.x'

// VTN Ads Library
implementation 'com.github.devvtn:vtn_ads_libs:x.x.x'
implementation 'com.facebook.shimmer:shimmer:x.x.x'
```
📖 **Giải thích:**
- `play-services-ads`: **SDK chính thức của Google AdMob, dùng để hiển thị quảng cáo Google.**
- `facebook-mediation`: **Kết nối quảng cáo Facebook vào hệ thống AdMob mediation (hiển thị xen kẽ).**
- `facebook-android-sdk`: **SDK Facebook cơ bản, cần cho đăng nhập, token, và ads.**
- `vtn_ads_libs`: **Thư viện tuỳ chỉnh (wrapper) giúp bạn dễ cấu hình quảng cáo.**
- `shimmer`: **Hiệu ứng “loading” lấp lánh (khi quảng cáo chưa tải xong).**

### Các loại quảng cáo
- ✅ **App Open Ads** - Hiển thị khi mở hoặc resume app
- ✅ **Interstitial Ads** - Quảng cáo toàn màn hình
- ✅ **Native Ads** - Tích hợp vào UI XML, Load ads native thành công sẽ add ads vào view
- ✅ **Banner Ads** - Quảng cáo banner ads (thường thấy ở layout bottomsheet của màn HOME)
- ✅ **Native Full Screen** - Load thành công Inter ADS sẽ hiện Native full ads toàn màn hình ngay sau đó

📖 **Giải thích:**
- `app_open_ads`: **Hiển thị khi người dùng mở hoặc quay lại ứng dụng (thường dùng cho màn Splash hoặc Resume).**
- `interstitial_ads`: **Quảng cáo toàn màn hình, thường xuất hiện sau khi người dùng hoàn thành một hành động (ví dụ: chuyển sang màn khác).**
- `native_ads`: **Quảng cáo tùy biến giao diện, hiển thị như một phần tự nhiên trong ứng dụng.**
- `banner_ads`: **Quảng cáo kích thước nhỏ, thường đặt ở cuối màn hình chính (trên thanh BottomTab).**
- `native_full_screen`: **Dạng native chiếm toàn màn hình, thường xuất hiện sau Interstitial Ads.**

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
📖 **Giải thích:**
- `repositories`: **Khai báo các kho (repositories) chứa thư viện để Gradle tải về.**
- `jitpack.io`: **Là nơi lưu thư viện `vtn_ads_libs` (vì đây là package GitHub, không có sẵn trên MavenCentral).**

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
📖 **Giải thích:**
- `INTERNET`, `ACCESS_NETWORK_STATE`: **Các quyền mạng cần thiết để tải và hiển thị quảng cáo.**
- `AD_ID`: **Quyền truy cập Advertising ID của thiết bị (yêu cầu từ Android 12 trở lên).**
- `<application>`: **Khai báo các tham số cấu hình trong thẻ `<application>` của AndroidManifest.**
- `APPLICATION_ID`: **ID ứng dụng trong Google AdMob, dùng để định danh app khi hiển thị quảng cáo.**
- `facebook_app_id`, `facebook_client_token`: **Cấu hình cho Facebook Ads SDK.**


### BƯỚC 3: Tạo file ads_id.xml

**File**: `app/src/main/res/values/ads_id.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Facebook -->
    <string name="facebook_app_id">YOUR_FB_APP_ID</string>
    <string name="facebook_client_token">YOUR_FB_CLIENT_TOKEN</string>
    <string name="adjust_token">YOUR_ADJUST_TOKEN</string>

    <!-- Google AdMob -->
    <string name="app_id">ca-app-pub-XXXXXXXX~XXXXXXXX</string>

    <!-- Real Ads IDs -->
    <string name="open_ads">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
    <string name="inter_splash">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
    <string name="native_all">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
    <string name="banner_all">ca-app-pub-XXXXXXXX/XXXXXXXX</string>
</resources>
```
📖 **Giải thích:**
- `ad_unit_ids`: **Lưu các mã ID quảng cáo thật hoặc test.**
- `App Open`, `Interstitial`, `Native`, `Banner`: **Mỗi loại quảng cáo có một mã ID riêng biệt.**
- `YOUR_FB_APP_ID`: **Thay bằng ID thật lấy từ Facebook Developer.**
- `adjust_token`: **Mã token của Adjust dùng để theo dõi và đo lường hiệu quả quảng cáo (nếu sử dụng Adjust SDK).**


### BƯỚC 4: Tạo Application Class

```kotlin
class MyApplication : AdsApplication() {

    override fun onCreate() {
        super.onCreate()
        RemoteConfig.init(this)

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
📖 **Giải thích chi tiết:**
- `Application`: **Báo cho hệ thống rằng đây là Application cấp cao nhất.**
- `class MyApplication : AdsApplication()`: **Khai báo Application của bạn kế thừa từ `AdsApplication` (lớp nền của thư viện VTN Ads).**
- `AdsApplication`: **Lớp nền của thư viện VTN Ads — khi kế thừa, bạn có thể `override` các hàm để tùy chỉnh hành vi quảng cáo.**
- `onCreate()`: **Hàm khởi tạo chính của Application, nơi gọi các khởi tạo liên quan đến ads/remote config/firebase.**
- `RemoteConfig.init(this)`: **Đọc file `ads_id.xml` và gán giá trị ID quảng cáo từ cấu hình remote/local.**
- `AppOpenManager.getInstance().disableAppResumeWithActivity(SplashActivity::class.java)`: **Tạm tắt hiển thị App Open Ads cho `SplashActivity` (tránh hiển thị quảng cáo quá sớm khi app resume).**
- `enableAdsResume()`: **Hàm override để bật/tắt tính năng App Open Ads khi app resume.**
- `getResumeAdId()`: **Hàm override trả về ID quảng cáo dùng cho App Open Ads.**
- `enableAdjustTracking()`: **Hàm override để bật/tắt theo dõi hiệu suất bằng Adjust (nếu sử dụng Adjust SDK).**
- `getAdjustToken()`: **Hàm override trả về token Adjust (lấy từ `ads_id.xml`).**
- `getKeyRemoteIntervalShowInterstitial()`: **Hàm override định nghĩa khoảng thời gian tối thiểu giữa hai lần hiển thị Interstitial Ads.**
- `getListTestDeviceId()`: **Hàm override trả về danh sách thiết bị test (ở ví dụ này là `null`).**
- `getIntentOpenNotification()`: **Hàm override trả về `Intent` được gọi khi người dùng mở app từ thông báo.**


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
📖 **Giải thích:**
- `RemoteConfig`: **Lớp chứa các biến toàn cục điều khiển việc hiển thị quảng cáo.**
- **Có 2 nhóm chính:**
    - **Biến bật/tắt quảng cáo:** `is_load_inter_splash`, `is_load_native_home`, ...
    - **ID quảng cáo:** `open_ads`, `inter_splash`, ... → lấy từ file `ads_id.xml`.
- `interval_show_interstitial`: **Biến quy định khoảng cách (tính theo giây hoặc số lần) giữa 2 lần hiển thị quảng cáo Interstitial.**
- `init(context)`: **Hàm đọc các chuỗi trong file `strings.xml` và gán giá trị tương ứng vào các biến trong RemoteConfig.**


### BƯỚC 6: Xin Consent (GDPR)

```kotlin
// Trong SplashActivity & MainActivity
private fun processAdConsent() {
    val consentHelper = ConsentHelper.getInstance(this)
    consentHelper.obtainConsentAndShow(this) {
        // Sau khi có consent, load ads
        loadAds()
    }
}
```
📖 **Giải thích:**
- `ConsentHelper`: **Hỗ trợ tuân thủ quy định GDPR của châu Âu.**
- **Mục đích:** Người dùng cần đồng ý (*consent*) cho phép ứng dụng thu thập dữ liệu quảng cáo cá nhân hoá.
- **Cơ chế hoạt động:**
    - Khi người dùng bấm **“Allow”**, callback `{ loadAds() }` sẽ được gọi để tiếp tục tải quảng cáo.
- **Ý nghĩa:** Bước này giúp ứng dụng hợp lệ khi phân phối quốc tế, đặc biệt tại khu vực **EU**.


---

## 💡 SỬ DỤNG QUẢNG CÁO

### 1. APP OPEN ADS

Tự động hoạt động thông qua `AppOpenManager`. Chỉ cần config trong Application class.

```kotlin
// Tắt App Open cho màn hình cụ thể
AppOpenManager.getInstance().disableAppResumeWithActivity(MainActivity::class.java)

// Bật lại trong Activity
AppOpenManager.getInstance().enableAppResumeWithActivity(MainActivity::class.java)
```
📖 **Giải thích:**
- `AppOpenManager`: **Tự động quản lý và hiển thị quảng cáo khi ứng dụng được mở hoặc resume.**
- **Cách sử dụng:**
    - **Tắt** ở `MainActivity` để tránh hiển thị quảng cáo trong lúc khởi tạo ứng dụng.
    - **Bật** lại ở `MainActivity` (màn hình chính) để quảng cáo hiển thị hợp lý khi người dùng quay lại app.



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
📖 **Giải thích:**
- `loadAndShowInter()`: **Hàm vừa tải vừa hiển thị quảng cáo Interstitial.**
- `nextAction()`: **Callback được gọi sau khi người dùng đóng quảng cáo — dùng để tiếp tục luồng xử lý, ví dụ như chuyển sang màn hình tiếp theo.**

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
📖 **Giải thích:**
- **Mục đích:** Tạo `FrameLayout` để làm container chứa quảng cáo **Native Ads**.
- **Khi load thành công:**
    - Inflate layout `native_ads_layout`.
    - Gắn quảng cáo vào `NativeAdView` thông qua hàm `pushAdsToViewCustom()`.
- **Khi load thất bại:** Ẩn `container` để tránh chiếm không gian trống trong giao diện.

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
📖 **Giải thích:**
- **Cơ chế preload:** Giúp quảng cáo được **tải sẵn trước**, nhờ đó có thể hiển thị **ngay lập tức** khi cần mà không phải chờ tải.
- **Dạng Singleton:** Đảm bảo quảng cáo chỉ được **load một lần duy nhất** cho toàn bộ ứng dụng, tránh tải lại nhiều lần gây lãng phí tài nguyên.

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
📖 **Giải thích:**
- `ConsentHelper.canRequestAds()`: **Kiểm tra xem người dùng đã đồng ý GDPR hay chưa.**
- `RemoteConfig.is_load_banner_all`: **Biến điều khiển việc hiển thị Banner Ads, được cấu hình từ xa (Remote Config).**
- `BannerPlugin.Config()`: **Đối tượng cấu hình cho Banner Ads.**
    - `defaultAdUnitId`: **ID quảng cáo banner được lấy từ file `ads_id.xml`.**
    - `defaultBannerType`: **Loại banner — ví dụ `Adaptive` (tự co giãn theo kích thước màn hình).**
- `loadBannerPlugin(...)`: **Hàm tải và chèn banner vào layout.**
- `binding.loBanner.visibility = View.VISIBLE`: **Hiển thị khu vực banner sau khi tải quảng cáo thành công.**

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
📖 **Giải thích:**
→ **Đảm bảo chỉ hiển thị quảng cáo khi thỏa các điều kiện sau:**
- **Quảng cáo được bật** trong `RemoteConfig`.
- **Người dùng đã cho phép** hiển thị (đồng ý điều khoản hoặc consent GDPR).
- **Thiết bị có kết nối mạng Internet.**
- **Đủ thời gian cách nhau** giữa hai lần hiển thị, được kiểm tra qua hàm `checkInterval()`.


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
📖 **Giải thích:**
→ **Giúp tránh việc “spam quảng cáo”.**
- `saveLastShowTime()`: **Lưu lại thời điểm hiển thị quảng cáo gần nhất, để kiểm soát khoảng cách giữa các lần hiển thị.**
→ **Khi khởi động ứng dụng**, tiến hành **tải sẵn quảng cáo Native và Interstitial** để có thể hiển thị **ngay lập tức** khi cần, giúp trải nghiệm người dùng mượt mà hơn.

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
📖 **Giải thích:**
- **Mục đích:** Giúp **ProGuard** (trình nén và làm rối code khi build bản release) **không xoá hoặc đổi tên** các class quan trọng mà SDK quảng cáo cần sử dụng.
- `-dontwarn`: **Bỏ qua các cảnh báo không cần thiết** trong quá trình rút gọn và tối ưu mã.

---

## ⚠️ LƯU Ý QUAN TRỌNG

| ⚠️ | Nội dung                                                        |
|----|-----------------------------------------------------------------|
| **Test Ads IDs** | Chỉ dùng khi development, KHÔNG dùng ở production               |
| **Consent** | Luôn kiểm tra `ConsentHelper.getInstance(this).canRequestAds()` |
| **Interval** | Tùy chỉnh giây giữa 2 lần show Interstitial trên RemoteConfig   |
| **Internet** | Kiểm tra kết nối trước khi load ads                             |
| **Cleanup** | Destroy ads trong `onDestroy()`                                 |
| **Remote Config** | Dùng để bật/tắt ads từ xa mà không cần update app               |

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
📖 **Giải thích:**
- **Thư mục này** chứa **toàn bộ cấu trúc ứng dụng mẫu** minh họa cách **tích hợp quảng cáo (ads)** đúng chuẩn.
- **Phân tách môi trường:**
    - `develop` và `production` được tách riêng với **2 file `google-services.json`** khác nhau, giúp quản lý cấu hình Firebase và AdMob theo từng môi trường.

---

## 🚨 TROUBLESHOOTING NHANH

| Vấn đề | Giải pháp                                                                                  |
|--------|--------------------------------------------------------------------------------------------|
| Ads không hiển thị | Kiểm tra remote config, consent, internet, Ad IDs hoặc máy ảo thường không show native ads |
| Crash khi show ads | Kiểm tra context null, Activity destroyed                                                  |
| Facebook mediation không work | Kiểm tra Facebook App ID, Client Token                                                     |
| Proguard error | Thêm keep rules cho ads classes                                                            |

---

**🎉 Hoàn tất! Bắt đầu kiếm tiền từ quảng cáo!**

