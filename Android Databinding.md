## Android Databinding - 1. 기초

HOXY..... RecyclerView ViewHolder 코드 이렇게 생기진 않았나요...?

```kotlin
class ViewHolder(holderView: View) : RecyclerView.ViewHolder(holderView) {
    private val textAccount: TextView = holderView.findViewById(R.id.text_account)
    private val textName: TextView = holderView.findViewById(R.id.text_name)
    private val textBio: TextView = holderView.findViewById(R.id.text_bio)
    private val textEmail: TextView = holderView.findViewById(R.id.text_email)
    private val textBlog: TextView = holderView.findViewById(R.id.text_blog)
    private val textInstragram: TextView = holderView.findViewById(R.id.text_instragram)
    private val textTwitter: TextView = holderView.findViewById(R.id.text_twitter)
    private val textGithub: TextView = holderView.findViewById(R.id.text_github)

    fun bind(profile: Profile) {
        this.textAccount.text = profile.account
        this.textName.text = profile.name
        this.textBio.text = profile.bio
        this.textEmail.text = profile.email
        this.textBlog.text = profile.blog
        this.textInstragram.text = profile.instragram
        this.textTwitter.text = profile.twitter
        this.textGithub.text = profile.github
    }
}
```

아.... 끔찍하군요. 😱😱😱😱😱



하지만 데이터바인딩을 쓴다면 ?

```kotlin
inner class ViewHolder(
    private val binding: ItemProfileBinding
) : RecyclerView.ViewHolder(binding.root) {

    fun bind(profile: Profile) {
        binding.profile = profile
    }
}
```

와우😮😮😮 단 몇 줄로 처음 코드와 동일한 기능을 하게 만들 수 있습니다! 😎

 



### 왜 데이터바인딩?

왜 데이터 바인딩을 사용해야 할까요?

위에 처럼 그저 코드량을 줄이기 위해서?



다른 예시를 들어보겠습니다.

```kotlin
findViewById(R.id.text_name)
```

위 코드는 View를 찾아오는 코드입니다.

이 코드만 보고 어떤 뷰를 찾아오는지 예상이 되시나요?

물론 id값을 잘 써놨다면, 충분히 유추가 가능합니다.

#### 하지만 그렇지 않다면?

알고보니 R.id.text_name이 ImageView 였다면??? 😱

```kotlin
private val textName: TextView = holderView.findViewById(R.id.text_name)
```

불러온 실제 View는 ImageView이기 때문에 Type이 맞지 않아 런타임상에서 앱이 터지게 될 것입니다.

이로 인해 findViewById는 **"Type Safe"** 하지 않다는 사실을 알 수 있습니다.



#### 한 가지 더 있습니다.

R.java 파일에는 안드로이드 리소스 id 값들이 모두 들어가 있습니다. xml 파일이 다르다고 다른 R파일에 id값이 할당되는 모양새가 아닙니다.

이 말은, 내가 생각하고 있는 View에 들어있는 View id값이 아닌 다른 파일에 있는 View Id를 접근할 수 있고, 그렇게 해도 컴파일러는 감지해내지 못한다는 뜻이 됩니다.

findViewById 호출 시에 현재 뷰에 존재하지 않는 id값을 사용한다면? 😱😱

```kotlin
private val textName: TextView = holderView.findViewById(R.id.somthing_other_file_view)
```

View에는 해당 id를 가지고 있는 View가 존재하지 않기 때문에 null을 반환하게 될 것입니다.

null인 View에 접근한다면 NPE(Null Point Exception)가 발생하게 되어 앱이 터지게 될 것입니다.

위 예시에서 findViewById는 "Null Safe" 하지 않다는 사실을 알 수 있습니다.



### Type Safe하고 Null Safe한 DataBinding

Databinding은 Binding 객체를 만들고, 해당 객체를 통해서 View에 접근합니다.

```kotlin
val binding: ItemProfileBinding = ItemProfileBinding.inflate(layoutInflater)
```

이런 식으로 객체를 생성하고,

```kotlin
binding.textGithub.text = "https://github.com/nightmare73"
```

이런식으로 View에 접근합니다.



xml에 선언 되어있고, id가 부여된 모든 View에 대해서는 binding 객체에서 모두 접근이 가능합니다.

그렇지 않은 View에 대해서는 애초에 접근이 불가능하기 때문에 null인 View는 존재하지 않습니다.

또, xml을 가지고 Android Studio가 Binding class를 만들기 때문에 Type이 다른 이상한 View를 내놓지 않습니다.



#### 두 줄 요약

* findViewById는 **Type Safe** 하지 않고, **Null Safe** 하지 않다!
* Databinding은 **Type Safe** 하고, **Null Safe** 하다!





## 데이터 바인딩 사용법

1. app단의 build.gradle에 아래 코드를 추가하고 **Gradle Sync**를 잊지 맙시다!

```groovy
plugins {
	...
    id 'kotlin-kapt'
}

android {
    ...
    buildFeatures {     
        dataBinding true
    }      
}
```



2. databinding을 사용하고 싶은 layout xml 파일에 다음과 같이 **\<layout\>** 태그를 가장 상위에 붙여줍니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout> <!-- 가장 상위에 적어주어야 합니다! -->
    <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        ...
        
        <TextView
            android:id="@+id/text_account"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="TextView"
            app:layout_constraintStart_toEndOf="@+id/image_profile"
            app:layout_constraintTop_toTopOf="@+id/image_profile" />
        
		...
        
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```



3. 마지막으로 레이아웃을 그렸으니 액티비티에 알려줘야겠죠?

Databinding으로 레이아웃을 inflate 시키는 방법은 여러가지가 있지만, 두 가지만 보여드리겠습니다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val binding = ActivityMainBinding.inflate(layoutInflater)
		setContentView(binding.root)
        
        binding.textAccount.text = "nightmare73@naver.com"
    }
}
```

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val binding: ActivityMainBinding =
            DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        binding.textAccount.text = "nightmare73@naver.com"
    }
}
```

이제 binding 객체를 통해 id가 등록된 View에 접근이 가능해집니다!



### 🛑Binding 객체를 찾는 방법🛑

layout xml 파일 이름을 자세히 봐주세요! (Binding은 자동으로 붙습니다.)

![databinding_binding_name](image\databinding_binding_name.PNG)



### 데이터 객체와의 결합

위 방법까지 따라오셨다면, 사실 의문점이 들 수 밖에 없습니다.

> 않이... 이러면 View에 접근해서 일일이 데이터 넣는거랑 뭐가달라..? 코드 못 줄이는건 똑같은데?

맞습니다!! 아직 안알려드렸기 때문이죠.



먼저, View에 보여주고 싶은 데이터들을 뭉쳐서 하나의 클래스로 만들어줍니다.

```kotlin
data class Profile(
    val account: String,
    val name: String,
    val bio: String,
    val email: String,
    val blog: String,
    val instragram: String,
    val twitter: String,
    val github: String,
)
```



다음으로 xml에 해당 Profile 객체를 쓰겠다는 선언을 해줍니다.

type은 Profi.... 까지만 쓰시고 기다리면 자동완성 되는걸 볼 수 있습니다. 만약 자동완성이 되지 않으면, 해당 객체가 위치한 패키지명을 정확하게 입력해주세요! 제 코드와 100% 다를 수밖에 없습니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>

    <data>

        <variable
            name="profile"
            type="com.malibin.example.databinding.Profile" />
    </data>
    
    <androidx.constraintlayout.widget.ConstraintLayout  xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        
        ...(중략)...
        
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```



다음으로는 xml에 직접 데이터를 엮어주는 작업입니다.

View xml의 속성에 @{} 라고 쓰면, "데이터바인딩으로 전달하겠다" 라는 의미가 됩니다. 

```xml
        <TextView
                  (나머지 속성들은 중략합니다)
            android:id="@+id/text_account"
            android:text="@{profile.account}"/>

        <TextView
            android:id="@+id/text_name"
            android:text="@{profile.name}" />
```

자, 마지막으로 Binding에 실제 Profile 객체를 넘겨주기만 하면 끝입니다!



```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        val profile = Profile(
            account = "nightmare73@naver.com",
            name = "Malibin",
            bio = "Hello",
            email = "nightmare73@naver.com",
            blog = "https://modelmaker.tistory.com/",
            instragram = "@malibin_",
            twitter = "",
            github = "https://github.com/nightmare73",
        )
        binding.profile = profile
    }
}

```



여기까지만 따라해도 데이터바인딩 반은 했습니다.

지금 내용으로는 text 속성 말고는 바꿀 수 있는게 별로 없는데요, 다음에는 다양한 Databinding에 대한 포스팅이 이어집니다.