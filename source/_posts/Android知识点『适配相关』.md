---
title: Android『适配相关』
tags:
---

##### Android ID

获取：

```java
String id = Settings.Secure.getString(mContext.getContentResolver(),
                 Settings.Secure.ANDROID_ID);
```

兼容问题：

In O, Android ID (Settings.Secure.ANDROID_ID or SSAID) has a different value for each app and each user on the device. Developers requiring a device-scoped identifier, should instead use a resettable identifier, such as Advertising ID, giving users more control. Advertising ID also provides a user-facing setting to limit ad tracking

Additionally in Android O:

- The ANDROID_ID value won't change on package uninstall/reinstall, as long as the package name and signing key are the same. Apps can rely on this value to maintain state across reinstalls.
- If an app was installed on a device running an earlier version of Android, the Android ID remains the same when the device is updated to Android O,<font color="#dd0000">**unless the app is uninstalled and reinstalled.**</font> 
- The Android ID value only changes if the device is factory reset or if the signing key rotates between uninstall and reinstall events.
- This change is only required for device manufacturers shipping with Google Play services and Advertising ID. <font color="#dd0000">**Other device manufacturers may provide an alternative resettable ID or continue to provide ANDROID ID.**</font>

##### startService



##### 动态权限



##### FileProvider



##### 定位权限

