---
title: Android Intro Activity - font family / text animation
category: android-ui
tags:
- AndroidUI
- AnroidAnimation
- Kotlin
- Android
---

참고자료   
[https://www.journaldev.com/9481/android-animation-example](https://www.journaldev.com/9481/android-animation-example)
[https://developer.android.com/training/animation/reveal-or-hide-view?hl=ko](https://developer.android.com/training/animation/reveal-or-hide-view?hl=ko)
[https://woovictory.github.io/2020/06/12/Android-Font/](https://woovictory.github.io/2020/06/12/Android-Font/)

<p float="center">
  <img src="/assets/images/gif/2020-10-2021-28.gif" width="300" />
</p>

* TextView를 추가하고 애니메이션 fade in 애니메이션 적용   
* custom font 다운받아 적용   
* Coroutine 맛보기   
* Activity 전환 애니메이션 적용   
* Ststus bar 크기를 구하고 로고의 위치 조정   

## res/anim/fadein.xml , fadeout.xml
fadein.xml
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="true">

    <alpha
        android:duration="1000"
        android:fromAlpha="0.0"
        android:interpolator="@android:anim/accelerate_interpolator"
        android:toAlpha="1.0" />
</set>
```
fadeout.xml
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="true" >

    <alpha
        android:duration="1000"
        android:fromAlpha="1.0"
        android:interpolator="@android:anim/accelerate_interpolator"
        android:toAlpha="0.0" />
</set>
```
* res 디렉토리에 anim 디렉토리를 추가하고 fadein.xml, fadeout.xml을 추가한다.   
* alpah는 물체의 불투명도(opacity)를 참조한다. 낮으면 투명해지고 높을수록 불투명하다.   
	* 애니메이션에서 fade란 알파값을 0에서 1로 변경하는것에 불과하다.   
	* fadeout은 정 반대로 지정하면 된다.   
* interpolator(보간)는 시작지점과 종료시점까지의 변화 과정을 어떤 식으로 표현할 것인가를 애니메이션으로 정의한 것이다.   
	* accelerate_interpolator는 시작지점 속도가 0으로 시작하여 점점 증가한다. 
	* accelerate_decelerate_interpolaotr (0부터 시작하여 증가했다가 마지막에 0으로 감소)  .. 등등   


## IntroActivity.kt 에 animation 구현   
onCreate() 내부
```
...
  var statusBarHeight:Int = 0;
  var resId = resources.getIdentifier("status_bar_height", "dimen", "android")
        
  if (resId > 0) {
    statusBarHeight = resources.getDimensionPixelSize(resId)
  }

  Log.e("barHeight", statusBarHeight.toString()) // result : 63 -> 디바이스에 따라 다를것으로 예상됨
				
  val introLogo = findViewById<TextView>(R.id.intro_logo)
  introLogo.setPadding(0,statusBarHeight,0,0)
...
```
* IntroActivity는 statusBar(상태바)를 띄우지 않고 MainActivity에서는 띄우게 된다.   
	* 이로인해 정 중앙에 위치하는 TextView 로고가 activity 이동시 아래로 내려가는 것 처럼 보이게 된다.   
	* 위 코드는 앱 시작시 상태바의 높이를 구하고 TextView에 Padding Top 에 입력하여 다음 화면과 동일하게 로고의 위치를 고정하기 위함이다.   

```
        val logoAnimation = AnimationUtils.loadAnimation(applicationContext,R.anim.fadein);
        introLogo.startAnimation(logoAnimation)
        
       CoroutineScope(Dispatchers.Main).launch {
           withContext(CoroutineScope(Dispatchers.Default).coroutineContext) {
               delay(4000L)
               val intent = Intent(this@IntroActivity, MainActivity::class.java)
               startActivity(intent)
               overridePendingTransition(R.anim.fadein, R.anim.fadeout)
               finish()
           }
       }
```
* AnimationUtils.loadAnimation() 메소드로 위에서 생성한 fadein.xml 을 호출한다.   
* startAnimation() 메소드에 입력하여 애니메이션을 시작한다.   
* delay를 위해 코루틴이 활용되었다.   4초뒤 MainActivity로 넘어가는 로직이다.   
* startActivity() 뒤에 overridePendingTransition() 메소드를 활용하면 액티비티 전환 애니메이션을 컨트롤할 수 있다.   
	* 위 로직은 fadeout을 구현하였다.    

## res/font/app_font.xml   
인터넷 상에서 무료로 사용할 수 있는 폰트를 다운받아 res/font 폴더에 저장하고 다음 xml을 생성한다.   
```
<?xml version="1.0" encoding="utf-8"?>
<font-family xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="UnusedAttribute,ResourceCycle">

    <font
        android:font="@font/gong_gothic_light"
        android:fontStyle="normal"
        android:fontWeight="300"
        app:font="@font/gong_gothic_light"
        app:fontStyle="normal"
        app:fontWeight="300" />

    <!--bold-->
    <font
        android:font="@font/gong_gothic_bold"
        android:fontStyle="normal"
        android:fontWeight="500"
        app:font="@font/gong_gothic_bold"
        app:fontStyle="normal"
        app:fontWeight="500" />

    <!--medium-->
    <font
        android:font="@font/gong_gothic_medium"
        android:fontStyle="normal"
        android:fontWeight="1000"
        app:font="@font/gong_gothic_medium"
        app:fontStyle="normal"
        app:fontWeight="1000" />

</font-family>
```
* 모든 폰트에 적용하는 방법과 styles.xml에 등록하여 필요한 곳에서만 사용하는 방법이 있다.    

res/values/styles.xml에 다음과 같이 등록해서 사용한다.   
```
    <style name="Text.RankStyle" parent="android:TextAppearance">
        <item name="android:textStyle">normal</item>
        <item name="android:textSize">14dp</item>
        <item name="android:textColor">@color/light_grey</item>
        <item name="android:fontFamily">@font/app_font</item>
        <item name="android:includeFontPadding">false</item>
    </style>

    <style name="Text.TitleStyle" parent="android:TextAppearance">
        <item name="android:textStyle">bold</item>
        <item name="android:textSize">16dp</item>
        <item name="android:textColor">@color/black</item>
        <item name="android:fontFamily">@font/app_font</item>
        <item name="android:includeFontPadding">false</item>
    </style>
```

TextView 의 `JroomUI` 로고 텍스트에 폰트를 적용해보자.   
```
    <TextView
        android:id="@+id/intro_logo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="@string/app_name"
        android:textSize="30sp"
        style="@style/Text.TitleStyle"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>
```
* `style="@style/Text.TitleStyle"` 처럼 적용하여 사용한다.
