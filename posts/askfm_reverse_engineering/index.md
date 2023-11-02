# Askfm Reverse Engineering: Getting Threads Back!


So you heared it. Askfm Reverse Engineering! to get back the threads to the web version by creating a web extension that calls the mobile api to show the data and more.
<!--more-->

# What the heck is askfm ?
Ah Askfm, the good old days when you get back from school and open your askfm account to check these question: which back at the time looked like 'What is your favourite colour ?, What order do you wear your shoes ? etc...'.

Askfm was something cute when almost few people (those who had opinios about everything) used it, now it became a monster, turned into an ad targeting application with many features that ruined the application : like coins (the first stupid thing), then many more stupid featues and many people used the application the wrong way (you know what I mean if you're using it now).

## Motivation
Probably one important feature which got nuked by askfm is : `threads`.
Back in time, `threads` was the best feature, you basically send a question to something with replies inside it, the reply isn't shown unless the owner publish it with answer. Replying with an image was a thing!

Now askfm nuked it, and made it only available for the mobile users (really!) with huge differences (it's called `chat` now):
- Anyone can reply to the question with anything without the permission of account owner!!!
- Can't reply with images as account owner.

Anyway, I am not here to convince you to leave askfm, I have a history of creating askfm bots and auto_repliers, when I logged into my very old account (which is deleted now btw) I tried to view a thread and I wasn't be able to do so (had to install the application on my phone).

So I thought it would be a great idea to have some sort of a web extension to just show the questions/replies under a question.

In this blog, I am sharing my journey to do so!

## Setup

After a quick research and inspection to the web version of the askfm, I found out that askfm mobile is different from the web version (ofc).

<br>
<p align="center">
    <img src="1.png" width="70%">
</p>
<br>

so at first glance, I thought about inspecting the traffic in the mobile application. So I tried many mobile application to inspect the traffic but nothing worked !!.

That's why I switched to using proxy applications like [charles](https://www.charlesproxy.com/) but it didn't work! (i don't remember why!).

Lastly found this awesome, game changer, program: [HTTPToolKit](https://httptoolkit.com/) but this need to run inside a VM, so I installed [Genymotion](https://www.genymotion.com/)(coz you know, I am lazy)

{{< admonition tip "Installing askfm on a VM" >}}
I wasn't be able to install the application on the VM on the first try to I had to do some tricks:
- I modified the buildprobs at : `/system/build.prop` and removed all the VM traces by Genymotion.
{{< /admonition >}}

### Inspecting the Traffic

As you can see below, The `API` askfm uses is mobile specific. and it uses `HMAC` for the authorization.

{{< image src="4.png" caption="HTTPToolKit in VM" >}}

`HMAC` (Hash-based Message Authentication Code) is a specific type of message authentication code that involves hashing a message with a `secret key` to produce a fixed-size output, which can be used to verify the integrity and authenticity of the message. `HMAC` is commonly used for authentication and integrity checking in various security protocols and applications, such as in web services, API authentication, and data transmission.

How HMAC authorization works is simple:

{{< image src="HMACDiagram.jpg" caption="How HMAC Works ?" >}}

So basically, there are 2 steps that the application does to send a `HTTP Request`:

1. <b>Key Generation</b> : The key should be in the app and the server agrees on using that key.
2. <b>HMAC Calculation</b>: The app calculates the HMAC by applying a specific cryptographic hash function (SHA-1) to the message and the secret key. The result of this calculation is a fixed-size hash value which is then send in the headers as : `Authorization: HMAC <hash>`

So all we have to do now it just find the `KEY` and how the app generates the `HMAC` hash! (easy?).

<br>
<p align="center">
    <img src="easy.jpg" width="50%">
</p>

### Decompiling the apk
So you're still here ?, good because we're just about to start the interesting part.
What's an `APK` you ask ? 
`APK` is a compressed archive file that contains all of the data and resources needed by an application to run on Android device. This mean we can extract it.

For this propose I used [Jadx](https://github.com/skylot/jadx) which is a command line and GUI tool for producing Java source code from Android Dex and Apk file, jadx can't decompile all 100% of the code, so errors will occur.

<br>

{{< image src="5.png" caption="Generating the hash" >}}

As you can see, I have found the part which generates the HMAC hash.

Digging the code more we'll find many interesting things: 

1. The needed headers (which are obvious from the network inspection).

- `Accept`, `Accept-Encoding`
- `X-Client-Type`: maybe used for client identification.
- `X-Api-Version`: the API version to use.
- `X-Access-Token`: the access token, which is fetched, somehow, separately.

```java
public final class RequestHeaders {
    public static final RequestHeaders INSTANCE = new RequestHeaders();

    private RequestHeaders() {
    }

    public final Map<String, String> headersWithSignature(String str, String accessToken) {
        Intrinsics.checkNotNullParameter(accessToken, "accessToken");
        TreeMap treeMap = new TreeMap();
        treeMap.put("Accept", "application/json; charset=utf-8");
        treeMap.put("Accept-Encoding", "identity");
        String clientType = Initializer.instance().getClientType();
        Intrinsics.checkNotNullExpressionValue(clientType, "instance().clientType");
        treeMap.put("X-Client-Type", clientType);
        String apiVersion = Initializer.instance().getApiVersion();
        Intrinsics.checkNotNullExpressionValue(apiVersion, "instance().apiVersion");
        treeMap.put("X-Api-Version", apiVersion);
        String hostWithPort = Initializer.instance().getHostWithPort();
        Intrinsics.checkNotNullExpressionValue(hostWithPort, "instance().hostWithPort");
        treeMap.put("Host", hostWithPort);
        treeMap.put("X-Forwarded-Proto", "https");
        if (!TextUtils.isEmpty(accessToken)) {
            treeMap.put("X-Access-Token", accessToken);
        }
        if (!TextUtils.isEmpty(str)) {
            StringCompanionObject stringCompanionObject = StringCompanionObject.INSTANCE;
            String format = String.format("HMAC %s", Arrays.copyOf(new Object[]{str}, 1));
            Intrinsics.checkNotNullExpressionValue(format, "format(format, *args)");
            treeMap.put("Authorization", format);
        }
        return treeMap;
    }
}
```

2. Server Params.
```java
public interface ServerParameters {
    public static final String ADVERTISING_ID_ENABLED_PARAM = "advertiserIdEnabled";
    public static final String ADVERTISING_ID_PARAM = "advertiserId";
    public static final String ADVERTISING_ID_WITH_GPS = "isGaidWithGps";
    public static final String AD_REVENUE_COUNTER = "adrevenue_counter";
    public static final String AD_REVENUE_PAYLOAD = "ad_network";
    public static final String AF_DEV_KEY = "appsflyerKey";
    public static final String AF_FIREBASE_TOKEN = "af_gcm_token";
    public static final String AF_USER_ID = "uid";
    public static final String AMAZON_AID = "amazon_aid";
    public static final String AMAZON_AID_LIMIT = "amazon_aid_limit";
    public static final String ANDROID_ID = "android_id";
    public static final String ANDROID_SDK_INT = "sdk";
    public static final String APP_ID = "app_id";
    public static final String APP_NAME = "app_name";
    public static final String APP_USER_ID = "appUserId";
    public static final String APP_VERSION_CODE = "app_version_code";
    public static final String APP_VERSION_NAME = "app_version_name";
    public static final String BATTERY_CHARGING_KEY = "btch";
    public static final String BATTERY_LEVEL_KEY = "btl";
    public static final String BRAND = "brand";
    public static final String CARRIER = "carrier";
    public static final String CHANNEL_SERVER_PARAM = "channel";
    public static final String CHECK_SUM1 = "cksm_v1";
    public static final String COUNTRY = "country";
    public static final String CURRENT_STORE = "af_currentstore";
    public static final String DDL_METRICS = "ddl";
    public static final String DEEP_LINK = "af_deeplink";
    public static final String DEEP_LINK_RESOLVED = "af_deeplink_r";
    public static final String DEFAULT_HOST = "appsflyer.com";
    public static final String DEFAULT_HOST_PREFIX = "";
    public static final String DEVICE_CURRENT_BATTERY_LEVEL = "batteryLevel";
    public static final String DEVICE_DATA = "deviceData";
    public static final String DEVICE_KEY = "device";
    public static final String DEVICE_TRACKING_DISABLED = "deviceTrackingDisabled";
    public static final String DEV_KEY = "devkey";
    public static final String EVENT_NAME = "eventName";
    public static final String EVENT_VALUE = "eventValue";
    public static final String FG_TS = "fg_ts";
    public static final String FIRST_LAUNCH = "firstLaunchDate";
    public static final String FIRST_LAUNCH_METRICS = "first_launch";
    public static final String FROM_FG = "from_fg";
    public static final String GCD_METRICS = "gcd";
    public static final String IMEI = "imei";
    public static final String INIT_TO_FG = "init_to_fg";
    public static final String INIT_TS = "init_ts";
    public static final String INSTALL_DATE = "installDate";
    public static final String INSTALL_STORE = "af_installstore";
    public static final String IS_BRANDED = "isBrandedDomain";
    public static final String IS_STOP_TRACKING_USED = "istu";
    public static final String LANG = "lang";
    public static final String LANG_CODE = "lang_code";
    public static final String LATEST_CHANNEL_SERVER_PARAM = "af_latestchannel";
    public static final String LAT_KEY = "lat";
    public static final String LAUNCH_COUNTER = "launch_counter";
    public static final String LOCATION_KEY = "loc";
    public static final String LON_KEY = "lon";
    public static final String META = "meta";
    public static final String MODEL = "model";
    public static final String NET = "net";
    public static final String NETWORK = "network";
    public static final String OAID = "oaid";
    public static final String ONELINK_ID = "onelink_id";
    public static final String ONELINK_VERSION = "onelink_ver";
    public static final String OPERATOR = "operator";
    public static final String ORIGINAL_AF_UID = "originalAppsflyerId";
    public static final String PAYLOAD_KEY = "payloadKey";
    public static final String PLATFORM = "platform";
    public static final String PLATFORM_EXTENSION = "platformextension";
    public static final String PREVIOUS_SESSION_DURATION = "prev_session_dur";
    public static final String PRE_INSTALL_NAME = "af_preinstall_name";
    public static final String REGISTERED_TO_UNINSTALL = "registeredUninstall";
    public static final String REINSTALL_COUNTER = "reinstallCounter";
    public static final String RETRIES = "retries";
    public static final String RFR_WAIT = "rfr_wait";
    public static final String SDK_DATA_SDK_VERSION = "sdk_version";
    public static final String STATUS = "status";
    public static final String STATUS_TYPE = "statType";
    public static final String TIMEOUT_VALUE = "timeout_value";
    public static final String TIMESTAMP = "af_timestamp";
    public static final String TIMESTAMP_KEY = "ts";
    public static final String TIME_PASSED_SINCE_LAST_LAUNCH = "timepassedsincelastlaunch";
    public static final String TIME_SPENT_IN_APP = "time_in_app";
    public static final String TOKEN_REFRESH_CONFIGURED = "tokenRefreshConfigured";
}
```

3. Request definitions for all the api routes.
```java
public enum RequestDefinition {
    TOKEN(0, "/token"),
    CONFIG(0, "/config"),
    REGISTER(1, "/register"),
    REGISTER_EXTERNAL(1, "/register/ext"),
    REGISTER_EXTERNAL_STATE(0, "/register/ext/state"),
    REGISTER_EXTERNAL_STATE_PUT(2, "/register/ext/state"),
    AUTHORIZE(1, "/authorize"),
    AUTHORIZE_EXTERNAL(1, "/authorize/ext"),
    AUTHORIZE_TWITTER(1, "/ext/authorize/twitter"),
    AUTHORIZE_VK(1, "/ext/authorize/vk"),
    AUTHORIZE_INSTAGRAM(0, "/ext/authorize/instagram"),
    CONNECT_INSTAGRAM(0, "/ext/connect/instagram"),
    CONNECT_INSTAGRAM_DELETE(3, "/connect/instagram"),
    CONNECT_EXTERNAL(1, "/connect/ext"),
    CONNECT_EXTERNAL_DELETE(3, "/connect/ext"),
    CONNECT_FACEBOOK_DELETE(3, "/connect/fb"),
    CONNECT_TWITTER(1, "/ext/connect/twitter"),
    CONNECT_VK(1, "/ext/connect/vk"),
    MY_PROFILE_GET(0, "/my/profile"),
    MY_PROFILE_PUT(2, "/my/profile"),
    MY_PROFILE_EMAIL_PUT(2, "/my/profile/email"),
    FRIENDS_PUT(2, "/friends"),
    FRIENDS_DEL(3, "/friends"),
    FRIENDS_ASK(0, "/friends/ask"),
    FRIENDS_MENTIONS(0, "/friends/mentions"),
    BLACKLIST_GET_FULL(0, "/blacklist/full"),
    BLACKLIST_DEL(3, "/blacklist"),
    SETTINGS_PASSWORD_CHANGE(1, "/settings/password/change"),
    SETTINGS_PASSWORD_RECOVER(1, "/settings/password/recover"),
    SETTINGS_PASSWORD_RESET(1, "/settings/password/reset"),
    SETTINGS_NOTIFICATIONS_GET(0, "/settings/notifications"),
    SETTINGS_NOTIFICATIONS_PUT(2, "/settings/notifications"),
    SETTINGS_SHARING_GET(0, "/settings/sharing"),
    SETTINGS_SHARING_PUT(2, "/settings/sharing"),
    MY_SHOUTOUTS_GET(0, "/my/shoutouts"),
    MY_QUESTIONS_GET(0, "/my/questions"),
    MY_QUESTIONS_THREAD_GET(0, "/my/questions/thread"),
    MY_QUESTIONS_THREAD_DELETE(3, "/my/questions/thread"),
    USERS_THREAD_GET(0, "/users/thread"),
    USERS_QUESTIONS(1, "/users/questions"),
    CHECK_NAME(0, "/users/check_name"),
    MY_QUESTIONS(3, "/my/questions"),
    MY_QUESTIONS_ANSWER(1, "/my/questions/answer"),
    MY_QUESTIONS_ANSWER_BACKGROUNDS(0, "/my/questions/answer/backgrounds"),
    MY_VIP_PROGRESS(0, "/my/vip/progress"),
    MY_DIRECT_MESSAGES(0, "/my/direct_messages"),
    DIRECT_MESSAGES_MARK_READ(2, "/direct_messages/mark_read"),
    LEADERS_FRIENDS(0, "/leaders/friends"),
    LEADERS_COUNTRY(0, "/leaders/country"),
    USERS_ANSWERS(0, "/users/answers"),
    MY_ANSWERS(3, "/my/answers"),
    USERS_ANSWERS_LIKES_GET(0, "/users/answers/likes"),
    USERS_ANSWERS_LIKES_PUT(2, "/users/answers/likes"),
    USERS_ANSWERS_LIKES_DELETE(3, "/users/answers/likes"),
    USERS_ANSWER_SHARE(2, "/users/answer/share"),
    USERS_INTERESTS(0, "/users/interests"),
    ANSWER_REWARDS_GET(0, "/rewards/answers"),
    NOTIFICATIONS(0, "/notifications"),
    COUNTERS(0, "/counters"),
    NOTIFICATIONS_MARK_READ(2, "/notifications/mark_read"),
    USERS_LIKES(0, "/users/likes"),
    USERS_GIFTS_GET(0, "/users/gifts"),
    USERS_DETAILS(0, "/users/details"),
    SEARCH_WITH_FRIENDS(0, "/users/search"),
    SEARCH(0, "/search"),
    SEARCH_FACEBOOK(0, "/search/fb"),
    SEARCH_PHONE_CONTACTS(0, "/search/phone_contacts"),
    SEARCH_FACEBOOK_COUNT(0, "/search/fb/count"),
    SEARCH_TWITTER(0, "/search/twitter"),
    SEARCH_VK(0, "/search/vk"),
    WALL(0, "/wall"),
    MY_PROFILE_SHARE_TWITTER(1, "/my/profile/share/twitter"),
    MY_DEVICE_DELETE(3, "/my/device"),
    MY_DEVICE_PUT(2, "/my/device"),
    UPLOAD_PHOTO(0, "/upload/photo"),
    UPLOAD_QUESTION_PHOTO(0, "/upload/question/photo"),
    UPLOAD_AVATAR_INIT(1, "/upload/avatar/init"),
    UPLOAD_AVATAR_FINISH(1, "/upload/avatar/finish"),
    UPLOAD_BACKGROUND_INIT(1, "/upload/background/init"),
    UPLOAD_BACKGROUND_FINISH(1, "/upload/background/finish"),
    UPLOAD_EXT(1, "/upload/ext"),
    REPORT_ANSWER(1, "/report/answer"),
    REPORT_USER(1, "/report/user"),
    REPORT_QUESTION(1, "/report/question"),
    FOLLOW_ALL_FB(2, "/follow/all/fb"),
    FOLLOW_ALL_TW(2, "/follow/all/tw"),
    FOLLOW_ALL_VK(2, "/follow/all/vk"),
    FOLLOW_ALL_PHONE_CONTACTS(2, "/follow/all/phone_contacts"),
    FOLLOW_SUGGESTIONS(0, "/follow/suggestions"),
    KARMA_WARNING_HIDE(2, "/karma/warning/hide"),
    BAD_ACTOR_WARNING_HIDE(2, "/badactor/warning/hide"),
    USERS_ACTIVATE(1, "/users/activate"),
    OPEN_EVENT(2, "/open"),
    GET_PYMK(0, "/pymk/suggestion"),
    DELETE_PYMK(3, "/pymk/suggestion"),
    ADD_TO_FAVORITES(2, "/friends/favorites"),
    REMOVE_FROM_FAVORITES(3, "/friends/favorites"),
    DEACTIVATE_ACCOUNT(1, "/users/disable"),
    ACCEPT_PRIVACY_POLICY(2, "/policy/accept"),
    UPLOAD_VIDEO(0, "/upload/video"),
    GALLERY_DELETE(3, "/my/profile/gallery/delete"),
    GALLERY_SET_AS_AVATAR(1, "/my/profile/gallery/setasavatar"),
    HASHTAG_SAVE(1, "/users/hashtags"),
    HASHTAG_REMOVE(3, "/users/hashtags"),
    HASHTAG_SEARCH(0, "/users/hashtags"),
    HASHTAG_USER_SEARCH(0, "/users/hashtags/search"),
    ANNOUNCEMENTS_GET(0, "/settings/announcements"),
    ANNOUNCEMENTS_PUT(2, "/settings/announcements"),
    PHOTO_POLLS_ITEM(0, "/photopolls"),
    PHOTO_POLLS_SAVE(1, "/photopolls"),
    PHOTO_POLLS_REMOVE(3, "/photopolls"),
    PHOTO_POLLS_REPORT(1, "/report/photopoll"),
    PHOTO_POLLS_VOTE(1, "/photopolls/vote"),
    PHOTO_POLLS_VOTERS(0, "/photopolls/users"),
    PHOTO_POLLS_UPLOAD(1, "/upload/photopoll"),
    PHOTO_POLLS_UPLOAD_EXTERNAL(1, "/upload/photopoll/ext"),
    PHOTO_POLLS_SHARE(2, "/users/photopoll/share"),
    PROFILE_STREAM(0, "/users/profile/stream"),
    ADS_TRACK(2, "/ads/track"),
    ADS_REWARD_VIDEO_POST(1, "/ads/videos"),
    ADS_REWARD_VIDEO_GET(0, "/ads/videos"),
    TRACK_VIEW_CARDS(2, "/track_view/cards"),
    TRACK_PAGE_VIEW(2, "/page_view"),
    TRACK_SOCIAL_DISMISS(2, "/track_view/soc_dismiss"),
    TRACK_REGISTRATION_ERROR(2, "/track_view/register/error"),
    TRACK_INSTALL(2, "/track/install"),
    TRACK_PHOTO_POLL_EVENT(2, "/track_view/photopoll/creation"),
    FETCH_USER_ID_SUGGESTIONS(0, "/suggest_username"),
    SHOUTOUT(1, "/shoutout"),
    SHOUTOUT_AVAILABLE(0, "/shoutout/available"),
    GEOIP(0, "/geoip/country"),
    TRACK_EVENTS(2, "/trackevents"),
    CHECK_CAPTCHA_REQUIRED(0, "/register/captcha_required"),
    INVITE_LINK(0, "/invites/link"),
    INVITE_OPEN(2, "/invite/open"),
    ME_PUT(2, "/me"),
    USERS_CONSENTS_GET(0, "/users/consents"),
    USERS_CONSENTS_PUT(2, "/users/consents"),
    TRENDING_USER_GET(0, "/users/trending"),
    SHARE_LINK_GET(0, "/users/share/link"),
    REWARD_ANSWERS_POST(1, "/rewards/answers"),
    MY_OFFERS_GET(0, "/my/offers"),
    MY_OFFERS_BUY_POST(1, "/my/offers/purchase"),
    VALIDATE_COINS_PURCHASE(1, "/purchase/coins/google/validate"),
    VALIDATE_SUBSCRIPTION_PURCHASE(1, "/purchase/subscriptions/google"),
    FINISH_PURCHASE(1, "/purchase/coins/google/finish"),
    UNLOCK_ANSWER(1, "/users/answers/unlock"),
    MY_DAILY_BONUS_POST(1, "/my/daily_bonus"),
    VIP_PROGRAM_WAIT_LIST(1, "/vip/program/waitlist"),
    VIP_PROGRAM_FORM(1, "/vip/program/form"),
    QUICK_INTRO_REPLIES_GET(0, "/quick_intro_reply/template"),
    SHOUTOUT_ONBOARDING_POST(1, "/shoutout/onboarding"),
    CHAT_GET(0, "/answers/chats"),
    CHAT_DELETE(3, "/answers/chats"),
    CHAT_MESSAGE_SEND_POST(1, "/chats/messages"),
    CHAT_MESSAGES_DELETE(3, "/chats/messages"),
    CHAT_MESSAGES_REPORT_POST(1, "/report/chats/messages"),
    PRIVATE_CHAT_GET(0, "/private_chats"),
    PRIVATE_CHAT_MESSAGE_SEND_POST(1, "/private_chats/messages"),
    PRIVATE_CHAT_DELETE(3, "/private_chats"),
    PRIVATE_CHAT_MESSAGES_DELETE(3, "/private_chats/messages"),
    PRIVATE_CHAT_REPORT_POST(1, "/report/private_chats"),
    MY_POPUPS_GET(0, "/my/popups"),
    MY_POPUPS_POST(1, "/my/popups"),
    SHOUTOUT_REBOARDING_CATEGORY_GET(0, "/shoutout/reboarding/categories"),
    SENT_SHOUTOUT_ANSWERS_GET(0, "/shoutouts/country/answers"),
    NO_REQUEST(0, "");
    
    public final String endpoint;
    private final int requestMethod;

    RequestDefinition(int i, String str) {
        this.requestMethod = i;
        this.endpoint = str;
    }

    public final int getRequestMethod() {
        return this.requestMethod;
    }
}
```

4. Parsing the response.

Note that `setNextRequestToken:  'X-Next-Token'` from the response. Also the timeDiff (probably some way to check the integrity of the request!)

```java
 @Override // com.android.volley.Request
    public Response<T> parseNetworkResponse(NetworkResponse response) {
        Response<T> error;
        Intrinsics.checkNotNullParameter(response, "response");
        RequestTokenStorage instance = RequestTokenStorage.Companion.instance();
        Map<String, String> map = response.headers;
        Intrinsics.checkNotNull(map);
        instance.setNextRequestToken(map.get("X-Next-Token"));
        TimeDiff timeDiff = this.timeDiff;
        Map<String, String> map2 = response.headers;
        Intrinsics.checkNotNull(map2);
        String str = map2.get("Date");
        Intrinsics.checkNotNull(str);
        timeDiff.setTimeDiffFromDate(str);
        try {
            byte[] bArr = response.data;
            Intrinsics.checkNotNullExpressionValue(bArr, "response.data");
            String parseCharset = HttpHeaderParser.parseCharset(response.headers);
            Intrinsics.checkNotNullExpressionValue(parseCharset, "parseCharset(response.headers)");
            Charset forName = Charset.forName(parseCharset);
            Intrinsics.checkNotNullExpressionValue(forName, "forName(charsetName)");
            String str2 = new String(bArr, forName);
            StringCompanionObject stringCompanionObject = StringCompanionObject.INSTANCE;
            String format = String.format("Finished %s %s with response: %s", Arrays.copyOf(new Object[]{this.request.getRequestMethodString(), this.request.getEndpoint(), str2}, 3));
            Intrinsics.checkNotNullExpressionValue(format, "format(format, *args)");
            Logger.v("ASKNetwork", format);
            if (Intrinsics.areEqual(new JSONObject(str2).optString("error", ""), "")) {
                Object fromJson = this.gson.fromJson(str2, this.typeToMap);
                if (fromJson != null) {
                    try {
                        if (this.request.getCacheKey() != null) {
                            ResponseCacheManager.instance().saveCachedResponse(this.request.getCacheKey(), str2);
                        }
                    } catch (ClassCastException e) {
                        error = Response.error(new ParseError(e));
                    }
                }
                error = Response.success(fromJson, HttpHeaderParser.parseCacheHeaders(response));
            } else {
                error = Response.error(new ServerError(response));
            }
            Intrinsics.checkNotNullExpressionValue(error, "{\n            val jsonSt…)\n            }\n        }");
            return error;
        } catch (JsonSyntaxException e2) {
            Response<T> error2 = Response.error(new ParseError(e2));
            Intrinsics.checkNotNullExpressionValue(error2, "{\n            Response.e…(ParseError(e))\n        }");
            return error2;
        } catch (UnsupportedEncodingException e3) {
            Response<T> error3 = Response.error(new ParseError(e3));
            Intrinsics.checkNotNullExpressionValue(error3, "{\n            Response.e…(ParseError(e))\n        }");
            return error3;
        } catch (JSONException e4) {
            Response<T> error4 = Response.error(new ParseError(e4));
            Intrinsics.checkNotNullExpressionValue(error4, "{\n            Response.e…(ParseError(e))\n        }");
            return error4;
        }
    }
```

5. The Signature. 

Probably, this is the most interesting peice of code, yes you guessed (HMAC hash generation).

{{< gist AYehia0 01926df3692ed1990b38a2fe91d3a2b6 >}}

Playing around with code has revealed some nasty parts: 

```java

// This is where the KEY is stored!
this.apiPrivateKey = new KeyGenerator().clientKey("XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");

// it calls a native lib to somehow encrypt/modify the hash
public class KeyGenerator {
    public native String clientKey(String key);

    public KeyGenerator() {
        System.loadLibrary("ffmpcodec");
    }
}
```

### Hmm, what's now
So far so good, let's recap what we have achieved so far.

- We found the KEY used in the HMAC generation, but a native lib called `ffmpcodec` is used to (maybe) perform some string modifications on the KEY.
- The HMAC `generateHash` method uses custom `sha-1`.
- The required headers are `X-Access-Token`, `X-Api-Version`, `X-Client-Type`.

