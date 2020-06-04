# Google Analytics / Google Firebase Analytics Fragment Tracking 



###   :chart: Google Firebase Analytics

​	Firebase SDK를 추가하면 screen view 이벤트를 자동으로 추적해준다. 단, **Acitivity** 만 자동으로 추적해준다 !!

​	하나의 Activity 안에 여러 Fragment가 존재하는 경우에도 화면 조회수를 분석하고 싶을 때에는 수동으로 screen view 이벤트를 호출 해주어야한다.

````kotlin
import android.content.Context
import androidx.fragment.app.Fragment
import com.google.firebase.analytics.FirebaseAnalytics

/**
 * Created By Yun Hyeok
 * on 2월 05, 2020
 */

open class TrackedFragment : Fragment() {
    
    override fun onAttach(context: Context?) {
        super.onAttach(context)

        FirebaseAnalytics.getInstance(context!!)
            .setCurrentScreen(activity!!, javaClass.simpleName, javaClass.simpleName)
    }
    
}
````

​	필자의 경우 위와 같이 추적이 되는 Fragment를 만들기 위해 클래스를 상속받고 Tracking 하고 싶은 Fragment에 상속 받게 하였다.

​	setCurrentScreen() 메소드를 이용하면 화면 추적이 가능하다. 화면 이름과 화면 클래스는 Activity가 변경되거나 setCurrentScreen()을 새로 호출할 때 까지 똑같이 유지된다고 한다.

​	Logcat에 아래와 같이 로그가 남는다면 수동 화면 추적이 잘 작동하고 있다는 의미이다.

````
Logging event (FE): screen_view(_vs), Bundle[{firebase_event_origin(_o)=auto, firebase_previous_class(_pc)=MypageFragment, firebase_previous_id(_pi)=-5900674759599319480, firebase_previous_screen(_pn)=MypageFragment, firebase_screen_class(_sc)=MainActivity, firebase_screen_id(_si)=-5900674759599319478}]

Logging event (FE): screen_view(_vs), Bundle[{firebase_event_origin(_o)=auto, firebase_previous_class(_pc)=MainActivity, firebase_previous_id(_pi)=-5900674759599319478, firebase_screen_class(_sc)=HomeFragment, firebase_screen_id(_si)=-5900674759599319477, firebase_screen(_sn)=HomeFragment}]
````



### :chart_with_upwards_trend: Google Analytics

* 구글 분석 툴을 이용하기 위해서는 아래와 같이 Dependency를 추가 해야한다.

````
implementation 'com.google.android.gms:play-services:12.0.1'
````



* 그 다음 Google Analytics 객체를 사용하기 위해 Application에 아래와 같은 코드를 작성한다.

````kotlin
class YourApplication : Application() {
    
    fun getTracker() : Tracker {
        try {
            val googleAnalytics = GoogleAnalytics.getInstance(this)
            return googleAnalytics.newTracker(R.xml.analytics)
        } catch (e: Exception) {
            Log.e(TAG, "Failed to initialize Google Analytics V4");
        }
        return null
    }
    
}
````



* Google Analytics 를 사용하기 위한 config 파일(Xml)을 작성한다.

````xml
<?xml version="1.0" encoding="utf-8" ?>
<resources xmlns:tools="http://schemas.android.com/tools" tools:ignore="TypographyDashes">

  <!--Replace placeholder ID with your tracking ID-->
  <string name="ga_trackingId">UA-XXXXXXXX-X</string>

  <!--Enable automatic activity tracking-->
  <bool name="ga_autoActivityTracking">true</bool>

  <!--Disable automatic exception tracking-->
  <bool name="ga_reportUncaughtExceptions">false</bool>

</resources>
````



* 추적을 원하는 Fragment에 아래의 코드를 상속받는다.

````kotlin
import android.content.Context
import androidx.fragment.app.Fragment

/**
 * Created By Yun Hyeok
 * on 2월 06, 2020
 */

open class TrackedFragment : Fragment() {

    override fun onAttach(context: Context?) {
        super.onAttach(context)

        val tracker = yourApplicationInstance.getTracker()
        tracker.setScreenName(javaClass.simpleName)
        tracker.send(HitBuilders.ScreenViewBuilder().build())
    }

}
````





### :stop_sign: 주의 사항 !! :stop_sign: 

이미 Firebase SDK를 사용중에 Google Analytics를 사용하기위해 gradle에 play-services를 추가한다면 클래스 중복으로 인한 컴파일 에러가 나타난다 !

````
Duplicate class com.google.android.gms.measurement.AppMeasurement found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-impl-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-impl:16.4.0)
Duplicate class com.google.android.gms.measurement.AppMeasurement$ConditionalUserProperty found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-impl-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-impl:16.4.0)
Duplicate class com.google.android.gms.measurement.AppMeasurement$Event found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-impl-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-impl:16.4.0)
Duplicate class com.google.android.gms.measurement.AppMeasurement$EventInterceptor found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-impl-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-impl:16.4.0)
Duplicate class com.google.android.gms.measurement.AppMeasurement$OnEventListener found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-impl-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-impl:16.4.0)
Duplicate class com.google.android.gms.measurement.AppMeasurement$Param found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-impl-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-impl:16.4.0)
Duplicate class com.google.android.gms.measurement.AppMeasurement$UserProperty found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-impl-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-impl:16.4.0)
Duplicate class com.google.firebase.analytics.FirebaseAnalytics found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-api-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-api:16.4.0)
Duplicate class com.google.firebase.analytics.FirebaseAnalytics$Event found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-api-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-api:16.4.0)
Duplicate class com.google.firebase.analytics.FirebaseAnalytics$Param found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-api-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-api:16.4.0)
Duplicate class com.google.firebase.analytics.FirebaseAnalytics$UserProperty found in modules jetified-firebase-analytics-impl-12.0.1-runtime.jar (com.google.firebase:firebase-analytics-impl:12.0.1) and jetified-play-services-measurement-api-16.4.0-runtime.jar (com.google.android.gms:play-services-measurement-api:16.4.0)

Go to the documentation to learn how to Fix dependency resolution errors.
````



#### 해결 방법

​	필자는 처음부터 Firebase를 이용하고 있었고, Firebase를 추후 Google Analytics에 연동하는 방법을 사용하였기 때문에 gradle에 play-services를 추가 하지 않고 Firebase Tracking을 통해 화면 조회수 추적을 수행 하였다.

​	이전 Firebase에서 가지고있던 통계들이 전부 Google Analytics 로 연동이 되기 때문에 이전 Firebase로 자동추적하던 조회수 또한 GA에서 통계치를 모두 보여주었다.





### 참고 링크들

> Google Firebase Analytics

https://stackoverflow.com/questions/45201346/how-to-track-android-fragments-using-firebase-analytics
https://firebase.google.com/docs/analytics/screenviews?hl=ko

> Google Analytics

https://stackoverflow.com/questions/14883613/using-google-analytics-to-track-fragments