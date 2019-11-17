# Kotlin Study (5/9) - 2019/11/05

Kotlin을 공부하기 위해 간단한 앱부터 복잡한 앱까지 만들어 봄으로써 Kotlin에 대해 하기!

총 9개의 앱 중 다섯 번째 앱

프로젝트명 : MyGallery

기능

* 기기에 저장된 사진을 차례대로 보여주고, 3초마다 자동으로 슬라이드 된다.
  

핵심 구성 요소

* Content Provider : 사진 정보를 얻을 때 사용한다.
  
* Fragment : UI의 일부를 나타낸다.
  
* ViewPager : 프래그먼트 여러 개를 좌우로 슬라이드하여 넘길 수 있게 해준다.
  
* FragmentStatePagerAdpater : 페이지가 많을 때 유용한 뷰페이지(ViewPager)용 어댑터이다.
  
* timer : 일정 시간 간격으로 반복되는 동작할 수 있다.

라이브러리 설정
* Anko : 인텐트 다이얼로그, 고르 등을 구현하는 데 도움이 되는 라이브러리
  
* Glide : 사진 로딩에 특화된 라이브러리로 메모리 절약과 자연스러운 사진 로딩에 사용한다.

## Content Provider 사용

앱의 데이터 접근을 다른 앱에 허용하는 컴포넌트이다.

프로바이더를 이용하여 사진 정보를 가지고 오는 순서이다.

1. 사진 데이터는 외부 저장소에 저장되어 있으므로 외부 저장소 읽기 권한을 앱에 부여한다.
2. 외부 저장소 읽기 권한은 위험 권한이므로 실행 중에 사용자에게 권한을 허용하도록 한다.
3. contentResolver 객체를 이용하여 데이터를 Cursor 객체로 가지고 온다.

>#### 안드로이드 저장소
> * `내부 저장소` : OS가 설치된 영역으로 유저가 접근할 수 없는 시스템 영역이다. 앱이 사용하는 정보와 데이터베이스가 저장된다.
> * `외부 저장소` : 컴퓨터에 기기를 연결하면 저장소로 인식되며 유저가 사용하는 영역이다. 사진과 동영상은 외부 저장소에 저장된다

>### 기기의 사진 경로 얻기

프로바이더를 사용해 사진 정보를 얻으려면 contentResolver 객체를 사용해 데이터를 얻을 수 있다. 아래 코드는 외부 저장소에 저장된 모든 사진을 최신순으로 정렬하여 Cursor라는 개체를 얻는 코드이다.

``` kotlin
val cursor = contentResolver.query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
            null, //가여올 항목 배열
            null, //조건
            null, //조건
            MediaStore.Images.ImageColumns.DATE_TAKEN + " DESC") //찍은 날짜 내림차순
```

그리고 Manifest에 권한을 추가해야 되는데, 안드로이드 6.0 버전부터는 모든 앱이 외부에서 리소스 또는 정보를 사용하는 경우 앱에게 사용자에게 권한을 요청해야 한다. 매니페스트에 권한을 나열하고 앱을 실행 중에 사용자에게 각 권한을 승인 받으면 된다.

> ### 권한 확인 및 요청

```kotlin

//권한이 부여되었는지 확인
        if(ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED){
            //권한이 허용되지 않음
            if(ActivityCompat.shouldShowRequestPermissionRationale(this,Manifest.permission.READ_EXTERNAL_STORAGE)) {
                //이전에 이미 권한이 거부되었을 때 설명
                alert("사진 정보를 얻으려면 외부 저장소 권한이 필수로 필요합니다.", "권한이 필요한 이유") {
                    yesButton {
                        //권한요청
                        ActivityCompat.requestPermissions(this@MainActivity, arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE),REQUEST_READ_EXTERNAL_STORAGE)
                    }
                    noButton {  }
                }.show()
            } else{
                //권한 요청
                ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE),REQUEST_READ_EXTERNAL_STORAGE)
            }
        } else{
            //권한이 이미 허용됨.
            getAllPhoto()
        }

```

위의 있는 코드는 실행 중에 위험 권한이 필요한 작업을 수행하게 될 때 권한이 있는지 확인하는 과정과 사용자에게 이 권한이 왜 필요한지를 알려주는 코드 및 권한을 승인 받도록 하는 코드이다.


>### 사용 권한 요청 응답 처리

```kotlin

  override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when(requestCode){
            REQUEST_READ_EXTERNAL_STORAGE -> {
                if((grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED)) {
                    getAllPhoto()
                } else{
                    toast("권한 거부됨.")
                }
                return
            }
        }
    }

```

위의 코드는 사용자가 권한을 요청하면 이 메서드를 호출하고 사용자의 응답을 전달하게 된다. 위의 코드에서 grantResults[0]는 위의 앱에서는 한개의 권한만 요청을 했기 때문에 0번 인덱스 값만 확인한 것이다.

만약 권한이 승인된다면 PERMISSION_GRANTED를 반환하고 거부했다면 PERMISSION_DENIED를 반환한다.

## Glide Library 사용

Glide 라이브러리는 미사용 리소스를 자동으로 해제하고 메모리를 효율적으로 관리 해주기 때문에 이미지를 보여줄 때 자주 쓰이는 라이브러리 이다. 그리고 이미지를 비동기로 로딩하여 UI의 끊김이 없다.

이 라이브러리를 사용하려면 build.gradle 파일에 의존성을 추가해야 한다.

```kotlin
dependencies {

    implementation 'com.github.bumptech.glide:glide:4.7.1'

}
```

Glide 사용 예제

```kotlin

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        Glide.with(this).load(uri).into(imageView)
    }

```

Glide.with(this)로 사용 준비를 하고 load() 메서드에 uri값을 인자로 주고 해당 이미지를 부드럽게 로딩한다. 이미지가 로딩되면 into() 메서드로 imageView에 표시한다. 이미지를 빠르고 부드럽게 로딩하고 메모리 관리까지 자동으로 하고 싶다면 Glide 라이브러리를 사용하는 것이 좋다. 코드도 한줄로 짧고 성능이 매우 좋다.

## MGallery_Kotlin을 통해 배운 것들

## 동반객체

프래그먼트는 특수한 제약 때문에 팩토리 메서드를 정의하여 인스턴스를 생성해야 한다. 팩토리 메서드는 생성자가 아닌 메서드를 사용해 객체를 생성하는 코딩 패턴을 말하는데 클래스와 별개로 보며 포함 관계도 아니다. 코틀린에서는 자바의 static과 같은 정적인 메서드를 만들 수 있는 키워드를 제공하지 않는다. 대신 동반 객체로 이를 구현한다.

사용예제

```kotlin

companion object {
        @JvmStatic
        fun newInstance(param1: String, param2: String) =
            PhotoFragment().apply {
                arguments = Bundle().apply {
                    putString(ARG_URI,uri)
                }
            }
    }

```

## Kotlin Study List

1. [BmiCalculator](https://github.com/hkd0694/BmiCalc_Kotlin)
2. [StopWatch](https://github.com/hkd0694/StopWat_Kotlin)
3. [MyWebBrowser](https://github.com/hkd0694/MyWeb_Kotlin)
4. [TiltSensor](https://github.com/hkd0694/TSens_Kotlin)
5. [MyGallery](https://github.com/hkd0694/MGallery_Kotlin)
6. [GpsMap](https://github.com/hkd0694/GpsMap_Kotlin)
7. [Flashlight](https://github.com/hkd0694/FLight_Kotlin)
8. [Xylophone](https://github.com/hkd0694/Xyloph_Kotlin)
9. [Todo 리스트](https://github.com/hkd0694/TodoList_Kotlin)