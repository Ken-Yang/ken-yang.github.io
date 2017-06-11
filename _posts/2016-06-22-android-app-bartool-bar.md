---
layout: post
title: "Android: App bar/Tool bar"
description: ""
category: 
tags: [Android, actionbar, appbar, toolbar, meterial design]
---
{% include JB/setup %}



大概一年多沒有寫Android了，最近發現好多不一樣了，<br />
譬如說ActionBar變成AppBar..不過以前的ActionBar還能繼續用，<br />
但還是得學一下怎麼用App bar跟上潮流。

[Example Code on Github] (https://github.com/Ken-Yang/AndroidAppBar)


<br />

---
### 1. Create new project
---

首先先建立一個新的project，然後選擇Empty activity。

<br />

---
### 2. Add dependency
---

接著打開build.gradle，把appcompat加入dependency，<br />
不過如果你的Android Studio是新版的，就會自動幫你加入了。

```xml
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'
}
```

<!--more-->

<br />

---
### 3. Remove actionbar
---

因為Default style會去繼承ActionBar Theme，所以我們要把它移除掉，<br />
打開`style.xml`，然後把parent改成`Theme.AppCompat.Light.NoActionBar`。

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
</style>
```

<br />

---
### 4. Color your App Bar
---

接著要去設定App Bar的顏色，AppBar主要有兩種顏色：

1. **colorPrimary** : 以前ActionBar那條的背景顏色
2. **colorPrimaryDark** : Status 那條的背景顏色

<br />

首先要先在`color.xml`中，新增上述的二個顏色。

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#FF5722</color>
    <color name="colorPrimaryDark">#E64A19</color>
</resources>
```

接著在`style.xml`中指定上述的二個顏色。

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
</style>
```

<br />

---
### 5. Create a toobar
---

接著去`res/layout`資料夾中，建立一個名稱為`too_bar.xml`的layout file，內容如下，有幾個需要說明的屬性為：

1. **theme**: 指定了Dark.ActionBar，是為了設定AppBar的一些style，譬如說字體顏色，Dark.ActionBar的字體顏色就是白色
2. **popupTheme**: 當你點擊ActionItem時，會popup一個list，這個list的theme
3. **background** : Toolbar的背景顏色

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
    android:background="@color/colorPrimary">

</android.support.v7.widget.Toolbar>
```

<br />

---
### 6. Add the toolbar into activity
---

然後要在主要的layout中，include上面的`tool_bar.xml`。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.ag888.ams.LoginActivity">

    <include
        android:id="@+id/tool_bar"
        layout="@layout/tool_bar"/>

    <TextView
        android:layout_below="@+id/tool_bar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />
</RelativeLayout>

```

<br />

---
### 7. Setup toolbar as actionbar
---

完成上面步驟以後，你可以試著run看看，但你會發現就只是簡單的一條bar而已，<br />
並沒有title或者action item之類的，所以接下要把toolbar設定成ActionBar，<br />
把`ToolBar`丟進去`setSupportActionBar()`就可以了。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_login);

    toolbar = (Toolbar) findViewById(R.id.tool_bar);
    setSupportActionBar(toolbar);
}
```

<br /><br /><br /><br />


