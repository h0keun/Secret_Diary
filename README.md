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

💡 SharedPreference 에서 commit()과 apply()의 차이는??? 
