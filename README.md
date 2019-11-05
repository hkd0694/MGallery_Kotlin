# Kotlin Study (5/9) - 2019/11/05

Kotlin을 공부하기 위해 간단한 앱부터 복잡한 앱까지 만들어 봄으로써 Kotlin에 대해 하기!

총 9개의 앱 중 네 번째 앱

프로젝트명 : TiltSensor

기능

* 기기를 기울이면 수평을 측정할 수 있다. 화면에 표시되는 원이 가운데로 이동하면 수평이다.
  

핵심 구성 요소

* SensorManager : 센서 관리자
  
* SensorEventListener : 센서 이벤트를 수신하는 리스너
  
* 커스텀 뷰 : 나만의 새로운 뷰를 만드는 방법

# Sensor 사용


>### Sensor 준비

안드로이드가 제공하는 센서 매니저 서비스 객체를 사용하면 센서를 사용할 수 있다. SensorManager는 안드로이드 기기의 각 센서 접근 및 리스너의 등록 및 취소, 이벤트를 수집하는 방법을 제공한다.

장치에 있는 센서를 사용하려면 getSystemService() 메서드를 이용한다.

```kotlin

class MainActivity : AppCompatActivity() {

    private val sensorManager by lazy {
        getSystemService(Context.SENSOR_SERVICE) as SensorManager
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
    }

}

```

> ### Sensor 등록

SensorManager 객체가 준비되면 액티비티가 동작할 때만 센서가 동작을 해야 배터리를 아낄 수 있기 때문에 액티비티가 사용자에세 화면을 보여주기 직적에 불리는 함수인 OnResume() 메서드에서 불러준다.

```kotlin

override fun onResume() {
        super.onResume()
        sensorManager.registerListener(this, sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER),
            SensorManager.SENSOR_DELAY_NORMAL)
        //SENSOR_DELAY_FASTEST  : 가능한 가장 자주 센서값을 얻는다.
        //SENSOR_DELAY_GAME     : 게임에 적합한 정도로 센서값을 얻는다.
        //SENSOR_DELAY_NORMAL   : 화면 방향이 전환될 때 적합한 정도로 센서값을 얻는다.
        //SENSOR_DELAY_UI       : 사용자 인터페이스를 표시하기에 적합한 정도로 센서값을 얻는다.
    }

```

>### Sensor Listener 등록

```kotlin


class MainActivity : AppCompatActivity(),SensorEventListener {

    //센서 정밀도가 변경되면 호출된다.
    override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {

    }

    //센서 값이 변경되면 호출된다.
    override fun onSensorChanged(event: SensorEvent?) {
        //센서값이 변경되면 호추로디는 메서드
        // value[0] : x 축 값 : 위로 기울이면 -10~0, 아래로 기울이면 0~10
        // value[1] : y 축 값 : 왼쪽으로 기울이면 -10~0, 오른쪽으로 기울이면 0~10
        // value[2] : z 축 값 : 미사용
        event?.let {
            Log.e("Start","OnSensorChanged x : + ${event.values[0]}, y : ${event.values[1]}, z : ${event.values[2]}")
        }

        tiltView.onSensorEvent(event)
    }
}

```

>### Sensor 해제

액티비티가 동작 중일 때만 센서를 사용하려면 화면이 꺼지기 직전인 onPause() 메서드에서 센서를 해제한다.

```kotlin
override fun onPause() {
    super.onPause()
    sensorManager.unregisterListener(this)
}

```

## TSens_Kotlin을 통해 배운 것들

### let() 함수

let() 함수는 블록에 자기 자신을 인수로 전달하고 수행된 결과를 반환한다. 인수로 전달한 객체는 it으로 참조한다. let()함수는 안전한 호출 연산자인 ?. 와 함께 사용하면 null값이 아닐 때에만 실행 할 수 있는 코드를 작성할 수 있다.

```kotlin

val result = str?.let{
    Integer.parseInt(it)
}
```
위 코드는 str이 null이 아닐 때만 정수로 변경하여 출력을 하는 코드이다. 복잡한 if문을 대체 할 수 있다.

TSens_Kotlin 사용 예제

```kotlin

    override fun onSensorChanged(event: SensorEvent?) {
        event?.let {
            Log.e("Start","OnSensorChanged x : + ${event.values[0]}, y : ${event.values[1]}, z : ${event.values[2]}")
        }
        tiltView.onSensorEvent(event)
    }
```



### lateinit, lazy 늦은 초기화

> #### lateinit

안드로이드를 개발할 때는 초기화를 나중에 해야 할 경우가 있는데, 이럴 경우 코틀린에서는 lateinit 키워드를 사용한다.

```kotlin

    lateinit car a : String

    a = "hello"
    println(a)
```

lateinit 사용 조건

* var 변수에서만 사용 가능 하다.
* null값으로 초기화할 수 있다.
* 초기화 전에는 변수를 사용할 수 없다.
* int, Long, Double, Float 에서는 사용 할 수 없다.

```kotlin

private lateinit var tiltView: TiltView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        tiltView = TiltView(this)
        setContentView(tiltView)
    }

```

위 코드에서는 tiltView 변수를 선언해주고 onCreate()가 실행될 때 TiltView인 커스텀 뷰를 만들어 주어 setContentView안에 tiltView를 넣어주는 코드이다.

> #### lazy

lateinit이 var로 선언한 변수의 늦은 포기화라면 lazy는 값을 변경할 수 없는 val을 사용할 수 있다. val 선언 뒤에 by lazy 블록에 초기화에 필요한 코드를 작성하면 된다.

```kotlin
val str:String by lazy{
    println("초기화")
    "Hello"
}
```
lazy로 늦은 초기화를 하면 앱이 시작할 때 연산을 분산시킬 수 있어 빠른 실행에 도움을 준다.

lazy 사용 조건

* val 변수에서만 사용 가능 하다.

조건에 적기 때문에 lateinit 보단 사용이 편리하다.

```kotlin

    private val sensorManager by lazy {
        getSystemService(Context.SENSOR_SERVICE) as SensorManager
    }

```
