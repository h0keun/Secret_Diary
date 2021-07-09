```💡 FastCampus 강의 수강 및 정리```

### Secret_Diary
+ ConstraintLayout
+ Custom Font
+ Handler : 요즘은 runOnUiThread 로 thread-runnable 내부에서 메인스레드로 접근시켜서 ui로 이어주는것이 가능함
+ SharedPreferences
+ Theme
+ AlertDialog

### 사용한 문법
+ android-ktx를 통한 SharedPreferences 사용하기(Kotlin Extension 인데 이거 이제 한물감)

### 기능
+ UI꾸미기
+ 비밀번호 저장, 변경 기능
+ 다이어리 내용 영구저장  
💡 실시간 저장 : Runnable, Handler, Looper 이용해서 5초 간격으로 자동저장
```KOTLIN
    private val handler = Handler(Looper.getMainLooper())
    ...
    private fun initDetailEditText() {
        val detail = getSharedPreferences("diary", Context.MODE_PRIVATE).getString("detail", "")
        diaryEditText.setText(detail)

        val runnable = Runnable {
            getSharedPreferences("diary", Context.MODE_PRIVATE).edit(true) {
                putString("detail", diaryEditText.text.toString())
            }
        }

        diaryEditText.addTextChangedListener {
            Log.d("DiaryActivity", "text Changed :: $it")
            handler.removeCallbacks(runnable)
            handler.postDelayed(runnable, 500)
        }
    }
```
[여기 참조해서 Thread-Handler-Runnable 쭉 확인해볼것](https://recipes4dev.tistory.com/143)

<img src="https://user-images.githubusercontent.com/63087903/119832555-3ea56580-bf39-11eb-98c5-32e845d158a9.jpg" width="200" height="430"> <img src="https://user-images.githubusercontent.com/63087903/119832547-3d743880-bf39-11eb-89b4-6fe4a2d05c96.jpg" width="200" height="430"> <img src="https://user-images.githubusercontent.com/63087903/119832552-3ea56580-bf39-11eb-8dd0-501aaf5a0f6f.jpg" width="200" height="430">

### [2021-04-23]

+ ConstraintLayout 속성중에 app:layout_constraintVertical_bias="0.45" 비율로 위치조정, chainStyle = packed로 붙임
+ NumberPicker에 테마 따로지정 themes.xml에서 따로 <style name= "??" 지정하고 <item name="android:textSize" 등 글자크기 글자색상 등 적용가능(요론 방법도 있다 정도)
+ xml에서 Button background 색상이 변하지 않아서 Button > androidx.appcompat.widget.AppCompatButton 으로 선언하고 색상 지정함
+ SharedPreference와 boolean 이용해서 요리조리 예외처리 해주면서 자물쇠 기능(Unlock, change passwort)구현
+ Handler-Runnable-SharedPreference 이용해서 작성중인 다이어리 내용이 5초동안 변화가 없으면 저장해줌
+ textChangedListener

### [2021-07-09]
#### manifest
+ Activity 추가
    ```KOTLIN
    * 두개의 액티비티를 사용하기 때문에 기존 MainActivity 이외에 추가한 엑티비티를 manifest에 선언
    <activity android:name="com.com.secret_diary.DiaryActivity"
        android:theme="@style/AppTheme.NoActionBar"/>
    ```
#### xml
+ 폰트지정
    ```KOTLIN
    * 임의의 폰트를 사용하기위해 폰트를 추가하였다.(배민 을지로10년후체)
    📂res ⁅
        📂 font ⁅
            ‣ bm_euljiro_10_years_later.ttf
     
    폰트를 지정하기 위해서는 원하는 텍스트뷰에
    android:fontFamily="@font/bm_euljiro_10_years_later"
    위의 속성을 추가하여 적용이 가능하다.
    ```
+ layout
    ```KOTLIN
    * activity_main.xml 과 activity_diary.xml - constraintlayout
    
    activity_diary는 전체화면을 EditText로 두었으며,
    activity_main은 자물쇠가 보이는 화면, activity_diary는 자물쇠를 열었을 때 다이어리 입력이 가능한 화면이다.
    메인화면에서는 NumberPicker를 3개 배치하여 각 자릿수 비밀번호를 Pick할 수 있도록 하였고, 
    다이어리 OPEN 을 위한 버튼 1개와 비빌번호 CHANGE를 위한 버튼 1개를 배치하였다.
    
    자물쇠의 위치는 화면 정중앙에서 살짝 위에 위치하도록 하였다. 이는 bias 속성으로 지정이 가능하다.
    app:layout_constraintVertical_bias="0.45"
    
    또한, 자물쇠 내부에 NumberPicker 들은 Contraint로 지정시 간격이 벌어지기 때문에 딱 붙이기위해서 chainStyle을 따로 지정해 주었다.
    app:layout_constraintHorizontal_chainStyle="packed"
    
    디테일한 contraintlayout 배치에대해서는 프로젝트의 소스코드를 참조하자!
    ```
#### Kotlin.class
+ MainActivity : 구현해야할 기능은 크게 2가지(자물쇠 열기, 비밀번호 변경하기)
    1.NumberPicker: 아래와 같이 minValue와 maxValue를 설정하는것이 가능하다.
    ```KOTLN 
    private val firstNumberPicker: NumberPicker by lazy {
        findViewById<NumberPicker>(R.id.firstNumberPicker)
            .apply {
                minValue = 0
                maxValue = 9
            }
    }
    ...
    ```  
    
   2.자물쇠를 열고 다이어리화면으로 넘어가기 : initOpenButton()
     엑티비티간 이동을 위해 intent를 사용한다. 이 프로젝트에선 intent가 동작해서는 안되는 경우가 두가지 있다.  
     [1. 비밀번호 변경중일 때 작동하지 않기], [2. 비밀번호가 틀렸을 때 작동하지 않기]
     ```KOTLIN
     private fun initOpenButton() {
        openButton.setOnClickListener {
            if (changePasswordMode) {
                Toast.makeText(this, "비밀번호 변경 모드입니다.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            val sharedPreferences = getSharedPreferences("password", Context.MODE_PRIVATE)
            val password = "${firstNumberPicker.value}${secondNumberPicker.value}${thirdNumberPicker.value}"

            if (password == sharedPreferences.getString("password", "000")) {
                startActivity(Intent(this, DiaryActivity::class.java))
            } else {
                showErrorPopup()
            }
        }
     }
     
     ...
     
     private fun showErrorPopup() {
        AlertDialog.Builder(this)
                .setTitle("실패")
                .setMessage("비밀번호가 잘못되었습니다.")
                .setPositiveButton("확인") { _, _ -> }
                .create()
                .show()
     }
     
      // changePasswordMode 는 전역인 Boolean 값으로 비밀번호 변경모드일때 다른화면으로 넘어가는것을 막아준다.
      // 비밀번호 변경중일 때는 true , 비밀번호 변경이 완료되면 false 
      // password는 각 자리 NumberPicker.value 를 통해 가져오게 되며,
      // 비밀번호 일치여부는 SharedPreferences를 통해 저장된 비밀번호와 비교하게된다.
      // 만약 비밀번호가 일치하지 않는다면 다이얼로그를 띄워준다.
     ```
     
  3.비밀번호 변경하기: initChangePasswordButton()  
    SharedPreferences를 이용해 비밀번호 초기값 000을 설정하고 changePasswordMode 가 true 인지 false 인지 에 따라 작동한다.  
    비밀번호 변경모드인지 아닌지에 따라 버튼의 색상을 다르게하여 UX를 업데이트한다.
    ```KOTLIN
    private fun initChangePasswordButton() {
        changePasswordButton.setOnClickListener {
            val sharedPreferences = getSharedPreferences("password", Context.MODE_PRIVATE)

            if (changePasswordMode) {

                sharedPreferences.edit {
                    this.putString("password", "${firstNumberPicker.value}${secondNumberPicker.value}${thirdNumberPicker.value}")
                    commit()
                }
                changePasswordMode = false
                changePasswordButton.setBackgroundColor(Color.BLACK)
            } else {
                val password = "${firstNumberPicker.value}${secondNumberPicker.value}${thirdNumberPicker.value}"

                if (password != sharedPreferences.getString("password", "000")) {
                    showErrorPopup()
                    return@setOnClickListener
                }

                changePasswordButton.setBackgroundColor(Color.RED)
                Toast.makeText(this, "변경할 비밀번호를 입력하고 다시 눌러주세요.", Toast.LENGTH_SHORT).show()
                changePasswordMode = true
            }
        }
    }
    ```
+ DiaryActivity  
  : 구현한 기능은 sharedPrefrence에 다이어리 내용을 저장하는 것인데,  
    이때 핸들러와 addTextChangedListener를 사용하여 다이어리 내용이 변경될 때마다 내용이 저장되도록 하였으며,  
    다이어리 내용이 변경될 때마다 매번 새로 저장하기에는 효율이 좋지 않아서 사용자가 텍스트 입력을 끝내고 
    0.5초동안 아무런 입력을 하지 않을 때 비로소 다이어리 내용을 저장하는 방식을 구현하였다.
    ```KOTLN
    class DiaryActivity : AppCompatActivity() {

        private val handler = Handler(Looper.getMainLooper()) // 메인 루퍼를 넣어주면 메인 스레드와 연결

        private val diaryEditText: EditText by lazy {
            findViewById<EditText>(R.id.diaryEditText)
        }

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_diary)

            initDetailEditText()
        }

        private fun initDetailEditText() {
            val detail = getSharedPreferences("diary", Context.MODE_PRIVATE).getString("detail", "")
            diaryEditText.setText(detail)

            val runnable = Runnable {
                getSharedPreferences("diary", Context.MODE_PRIVATE).edit(true) {
                    putString("detail", diaryEditText.text.toString())
                }
            }

            diaryEditText.addTextChangedListener {
                Log.d("DiaryActivity", "text Changed :: $it")
                handler.removeCallbacks(runnable) // 이전에 pending 되어 실행되지 않은 러너블이 있다면 제거하기 위함임
                handler.postDelayed(runnable, 500) //0.5초 이후에 실행
            }
        }
    }
    
    * 핸들러 객체 생성 시 메인 루퍼를 넣어 연결해주면 메인 스레드와 연결이 되기 때문에 UI 자원을 사용할 수 있게 된다.
    * 핸들러의 자동 저장에 사용될 Runnable 객체를 정의해주고 SharedPreferences 를 사용하여 다이어리 내용을 저장한다.
    * addTextChangedListener 에서 핸들러 자동저장을 구현하였는데 Runnable을 사용하며, postDelayed를 통해 0.5초 이후에 실행되도록 하였다.
    * 이 때 계속해서 text가 수정되어지면 해당 팬딩건이 계속 쌓일 수 있기 때문에 removecallbacks를 통해서 여태까지 쌓여있는 모든 runnable을 제거하도록 해준다.
    ```

💡 SharedPreference 에서 commit()과 apply()의 차이는??? 
```KOTLIN
commit()을 사용하면 변경시 UI가 정지되며 저장을 바로하지만 UI가 정지된다는 치명적인 문제가 있기 때문에 applay()를 통해 send message를 날리는 편이 더 선호되는것 같다.

이부분을 추가적으로 더 알아보쟈!
```

💡 lazy 를 이용한 늦은 초기화
```KOTLIN
메인 엑티비티가 생성될 때 뷰가 전부 다 생성된다는 것을 보장할 수 없기 때문에
뷰가 모두 생성되고나서 onCreate 내부에서 초기화를 진행할 수 있도록 
lazy 키워드를 사용한다.

ex)
class DiaryActivity: AppCompatActivity() {
    ...
    private val diaryEditText: EditText by lazy {
        findViewById<EditText>(R.id.diaryEditText)
    }
    ...
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_diary)
        
        ...
        diaryEditText ~~~
        ...
    }

위와 같이 lazy키워드를 사용 하였다면, 반드시 onCreate 내부에서 수동으로 코드를 추가하여 초기화 되도록 해야한다. 
이 프로젝트에서는 onCreate에서 함수를 호출하고 호출된 함수 내부에 diaryEditText를 추가해주었음!
```

💡 추가로 공부할 것! : runnable, thread 동작방식과 원리에대해서 세세하게 알아보기
