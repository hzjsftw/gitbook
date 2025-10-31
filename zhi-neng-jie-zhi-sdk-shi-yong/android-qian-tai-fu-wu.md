---
description: >-
  目前蓝牙连接服务是后台的，存在息屏状态下，或者app进入后台，蓝牙断连的问题，好处就是功耗低，坏处就是用户体验差，比如手势功能，切换到其他app，体验手势，切回app，就需要重连。如果需要将服务做成前台服务，可以在Application的onCreate()里设置
icon: square-font-awesome-stroke
---

# Android 前台服务

### 通知标题设置

```java
BLEUtils.contentTitle = "自己想要展示在app前台服务里的内容"
```

这个需要自己在合适的时机，比如app退到后台时，5分钟后，进行断连，可以参考《Android公版app的重连》

{% content-ref url="images-and-media.md" %}
[images-and-media.md](images-and-media.md)
{% endcontent-ref %}

&#x20;可以通过设置BLEUtils.pendingIntent，可以响应前台服务通知点击事件，如果app在后台，点击前台服务，跳转到指定页面。 例如：

```java
        // 创建返回首页的Intent
        Intent notificationIntent = new Intent(this, MainActivity.class);
        notificationIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TOP);
        notificationIntent.setAction(Long.toString(System.currentTimeMillis())); // 确保每次都是唯一的

        PendingIntent pendingIntent = PendingIntent.getActivity(
                this,
                0,
                notificationIntent,
                PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE
        );
        BLEUtils.pendingIntent=pendingIntent;
```

### 高版本android系统适配

在高版本的手机系统里，比如android 15，默认是不显示通知，需要手动申请权限

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

然后在代码里申请，尽量在连接蓝牙之前，服务是在蓝牙连接的service里开启的

```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS) != PackageManager.PERMISSION_GRANTED) {
    // 如果没有权限，向用户申请
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.POST_NOTIFICATIONS}, 1);
}
```

### 实时更新标题

有客户可能需要在前台服务更新一些变化的内容，比如电量变化之类，sdk也提供了方法，可以通过广播的形式，进行发送

**android:**

```java
Intent updateIntent = new Intent("ACTION_UPDATE_TITLE");
updateIntent.putExtra("EXTRA_NEW_TITLE", "新的标题");
sendBroadcast(updateIntent);
```

### 点击服务，跳转到页面

点击前台服务，默认是不跳转到任何页面的，如果想要添加，可以这样设置

```java

// 创建返回首页的Intent
Intent notificationIntent = new Intent(this, MainActivity.class);
notificationIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TOP);
notificationIntent.setAction(Long.toString(System.currentTimeMillis())); // 确保每次都是唯一的

PendingIntent pendingIntent = PendingIntent.getActivity(
        this,
        0,
        notificationIntent,
        PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE
);
BLEUtils.pendingIntent=pendingIntent;
```
