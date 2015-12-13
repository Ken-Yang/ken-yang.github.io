---
layout: post
title: "Android Fingerprint API"
description: ""
category: 
tags: [android, fingerprint]
---
{% include JB/setup %}

<font size=3>

Google在Android 6.0中，釋放了對[Fingerprint] (http://developer.android.com/intl/zh-tw/reference/android/hardware/fingerprint/FingerprintManager.html)操作的API，  
而最近剛好也買了Nexus 6P，因此有了機會來玩看看。



</br>

---
### 1. AndroidManifest.xml
---

首先要在`AndroidManifest.xml`中設定permission，

```xml
<uses-permission android:name="android.permission.USE_FINGERPRINT" />
```


</br>

---
### 2. Check Requirement
---

接著要檢查device上，是否有fingerprint reader，以及是否有設置了至少一枚fingerprint。下面這段code，主要檢查了三件事情：  

1. **isKeyguardSecure** : 是否有設定screen lock
2. **isHardwareDetected** : device是否有fingerprint reader
3. **hasEnrolledFingerprints** : 是否有設定至少一枚指紋

<!--more-->

```java
private KeyguardManager km;
private FingerprintManager fm;
private CancellationSignal cancellationSignal;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    km = (KeyguardManager)getSystemService(Activity.KEYGUARD_SERVICE);
    fm = getSystemService(FingerprintManager.class);

    if (!km.isKeyguardSecure()) {
       Toast.makeText(this,
                "Secure lock screen hasn't set up.\n"
                + "Go to 'Settings -> Security -> Fingerprint' to set up a fingerprint",
                Toast.LENGTH_LONG).show();
        return;
    }

    if (!isHardwareDetected()) {
        Toast.makeText(this,
                "No Fingerprint reader",
                Toast.LENGTH_LONG).show();
        return;
    }

    if (!fm.hasEnrolledFingerprints()) {
        Toast.makeText(this,
                "Go to 'Settings -> Security -> Fingerprint' and register at least one fingerprint",
                Toast.LENGTH_LONG).show();
        return;
    }
}
```



</br>

---
### 3. Authenticate
---

如果通過上述的檢查之後，就可以透過`FingerprintManager`進行`Authenticate`的動作，method call為：

```java
public void authenticate (FingerprintManager.CryptoObject crypto, 
                          CancellationSignal cancel, 
                          int flags, 
                          FingerprintManager.AuthenticationCallback callback, 
                          Handler handler)
```
一共要帶入5個參數，分別為：

1. `CryptoObject` : Android 6.0中crypto objects的wrapper class，可以透過它讓authenticate過程更為secure，但也可以不使用。
2. `CancellationSignal` : 用來Cancel *authenticate*的object
3. `flags` : 只是一個flag，且目前只能代入0
4. `AuthenticationCallback` : callback來接受authenticate成功與否，有三個callback method，
	* onAuthenticationError
	* onAuthenticationFailed
	* onAuthenticationSucceeded 
5. `Handler` : optional的參數，如果有使用，FingerprintManager會透過它來傳遞訊息

</br>

了解參數以後就可以開始進行**authenticate**，

```java

@Override
protected void onCreate(Bundle savedInstanceState) {
	.... ignore ...
	startListening();
}

private void startListening() {

    cancellationSignal = new CancellationSignal();

    fm.authenticate(null,
                    cancellationSignal,
                    0,
                    new FingerprintManager.AuthenticationCallback() {
                        @Override
                        public void onAuthenticationError(int errorCode, CharSequence errString) {
                            Log.e("", "error " + errorCode + " " + errString);
                        }

                        @Override
                        public void onAuthenticationFailed() {
                            Log.e("", "onAuthenticationFailed");
                        }

                        @Override
                        public void onAuthenticationSucceeded(FingerprintManager.AuthenticationResult result) {
                            Log.i("", "onAuthenticationSucceeded");
                        }
                    },
                    null);

}

@Override
public void onPause() {
    super.onPause();
    cancellationSignal.cancel();
    cancellationSignal = null;
}
```

</br>

這樣就完成了一個簡單的example，但這example並沒有使用`CryptoObject`，所以如果你想要更安全一點，記得請加上`CryptoObject`。


</br>




</font>