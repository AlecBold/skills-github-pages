---
title: Welcome to my blog
---

# HR
---
How to talk with HR - https://habr.com/ru/companies/sibur_official/articles/899720/

# Android Platform
---
## Application

Context - interface to global information about application environment (like a provider to the system); resources, preferences, launch activities, bind and launch, services, send broadcast, register broadcast receivers, layout inflation.

Activity Context vs Application Context - application cant do action that related to ui (launch activities, inflation) *(unless is new task for activity; application can inflate view, but it will be using default theme of the system not application’s)

## Components
Components are essential blocks of app. Each component is an entry point through which system or user can enter app.
- [[#Activity]]
- [[#Service]]
- [[#Content providers]]
- [[#Broadcast receivers]]

### Activity
Entry point for interacting with the user. Represents as a single screen with user interface. 

**Lifecycle**
* `onCreate()`: init main components of activity and setContentView(id); this method provides with a Bundle containing Activity’s previously state, if there was one
* `onStart()`: ui is visible
* `onResume()`: interface is accessible for a user
* `onPause()`: interface not accessible for a user
* `onStop()`: ui is not visible for user
* `onRestart()`: called if this activity coming back to interact with the user to; after this method onStart() will be called
* `onDestroy()`: get rid of all resources that activity was depended on; called when activity is finishing; either called after calling method finish() or because the system is destroying to save space

#### Fragments
Fragments can better modularize the code. It is reusable portion of app’s UI. Fragment defines and manages he’s own layout, has it’s own lifecycle, can handle it’s own input events.

##### Lifecycle
Each possible `Lifecycle` state is represented in the [`Lifecycle.State`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State) enum.
- [`INITIALIZED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#INITIALIZED)
- [`CREATED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#CREATED)
- [`STARTED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#STARTED)
- [`RESUMED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#RESUMED)
- [`DESTROYED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#DESTROYED)

**Callback methods in Fragment**
* onAttach()
* onCreate()
- onCreateView()
- onViewCreated()
- onActivityCreated()
- onViewStateRestored() - ?
- onStart()
- onResume()
- onPause()
- onStop()
- onSaveInstanceState()
- onDestroyView()
- onDestroy()
- onDetach()

![fragment-view-lifecycle.png|500](fragment-view-lifecycle.png)

### Service
Is a general purpose entry point for keeping app in background for all kinds of work. . Doesn’t provide UI. Another component can start service such as an activity and let it run  or bind to it to interact with it. Has 3 types:

**Started services** tell the system to keep them running until their work is done. This might be running data sync or playing music in the background even if after the user leaves the app. Playing music and syncing data is 2 different types of started services which the system handles differently.

- **Foreground**: ****Playing music is something important for user and need to be act as foreground through notification to tell the user that is still running. In this case the system is prioritizes keeping that service’s process running.
- **Background**: Regular background process isn’t that user is aware of. So the system has more freedom in managing its process. It might let it be killed and restarting this process sometime later, if it needs RAM.

To start service you need to call Context.startService(). Can be called multiple times, but onStartCommand() will be called multiple times. There are two additional modes depending on value returned by onStartCommand(): START_STICKY it is used for services that are explicitly started and stopped as needed, START_NOT_STICKY or START_REDELIVER_EVENT for services that should be remain running while processing commands sent to them. 

**Bound services** run because some other app or services has said that it wants to use of the service. A bound service provide API to another process so the system knows there’s dependency between them. So the system will treat bounded service’s process as a process that is using it.

Because of their flexibility, services are useful building blocks for all kinds of higher-level system concepts. Like: live wallpapers, notification listeners,  screen savers, input methods and many other core system features are built as services that applications implement and the system bind to it when they run.

Use Context.bindService() to persistent connection to a service. This creates service if it is not already running, calling onCreate() (but does not call onStartCommand()). The client will receive IBinder object that the service returns from from its onBind(Intent) method, allowing the client make calls back. 

Controlled through IBinder in AIDL(Android Interface Definition Language) or through Java object.

---

**Implementation**

Services can be started with Context.startService() or Context.bindService(). Run in the main thread of their hosting process(same as the application it is part of), it is not running in separate process unless otherwise specified. Multiple calls to startService() do not nest (though they do result in multiple corresponding calls to onStartCommand())

A service can be both **started** or **bounded**. In such case the system will keep the service alive as long as it is started or there are one or more connections to it.

Can communicate through Binder or Messanger to send Message objects(is just wrapper around Binder) also we can use AIDL(but i’ts complicated)

**Lifecycle**

will run until Context.stopService() or stopSelf()

![service_lifecycle.png|500](service_lifecycle.png)


#### JobScheduler #todo
[https://developer.android.com/reference/android/app/job/JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler)

### Broadcast receivers

Is component that lets the system deliver events to the apps outside of a regular user flow so the app can respond to system-wide broadcast announcements (*общесистемные широковещательные объявления*). It is also another entry point to the app. The system can also deliver broadcast even to app that is not currently running.

For example app can schedule an alarm to post notification to some future event. Because the alarm is delivered to BroadcastReceiver in the app, there is no need for an app to remain active. System broadcast: battery is low, picture is captured, screen turned off.  Apps can also initiate broadcast, for example let other apps know that some data is downloaded.

There are 2 types of Broadcast receivers:

**Static** is a broadcast that declared in manifest its receives events even when app is not currently active.

```xml
<receiver name=".SomeReceiver">
	<intent-filter>
		<action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
	</intent-filter>
</receiver>
```

**Dynamic** is a broadcast that don’t declared in manifest but in app for example in activity or app.

```kotlin
lateinit var broadcastReceiver: MyBroadcastReceiver
fun onCreate() {
	super.onCreate()
	broadcastReceiver = MyBroadcastReceiver()
	val intentFilter = IntentFilter()
	intentFilter.addAction(ConnectivityManager.CONNECTIVITY_ACTION)
	registerReceiver(broadcastReceiver, intentFilter)
}

fun onDestroy() {
	unregisterReceiver(broadcastReceiver)
}
// also broadcast receiver can be created with anonymous class, with this approach we can access app code (or really we can just set objects to constructor)
broadCastReceiver = object: BroadcastReceiver() {
	override fun onReceive(context: Context?, intent: Intent) {
		
	}
}
```

### Content providers
Content provider manages a shared set of app data that you can store in file system, database, web, or any persistent location that youre app can access. Through that, other apps can query or modify data, if content provider permits it.   

For example Android system has a content provider that manages the user’s contact information.

To the system, content provider is entry point into an app for publishing named data items, identified by URI scheme. App then get entities by URI address

TODO: add more info about content providers

## Startup application
### Architecture
1. Zygote 
2. System Server
3. Launcher

#### Zygote
Процесс который запускается вместе с системой. Он предзагружает в себя все основные классы Android Framework (из `frameworks/base.jar`) и общие ресурсы. 
Его задача — быстро форкать себя, чтобы создавать новые процессы приложений. Это экономит память (через Copy-on-Write) и ускоряет запуск.
#### System Server
Главный процесс системы (попрожден Zygote'ой). В нем работают сервисы:
* Android Manager Service (**AMS**) - диспетчер `Activity`, `Process`'s и `Tasks`
* Package Manager Service (**PMS**) - управляет информацией об установленных приложениях (`Manifest's`, `Permission's`, etc)
* Window Manager Service (**WMS**) - управляет окнами, поверхностями, вводом и композицией.
#### Launcher
Системное приложение через которое пользователь может запустить приложение нажав на  иконку.

### Process of launching

**Launcher**
1. [[#Launcher]] создает [[#Intent]] c `action=Main` и `category=LAUNCHER`, указывающий на `Activity` (указан в intent filter)
2. `startActivity(intent)` - так же вызывает [[#Launcher]]
3. в конечном сечете(`Activity -> ContextImpl`) вызов приводит к **Binder IPC-вызову** в [[#System Server]] (конкретно в **AMS**)

**System Server and Zygote**
1. Проверка разрешений и [[#Intent]]: **AMS** получает запрос и проверяет
	* существует ли такая **Activity** (через **PMS**)
	* есть ли у [[#Launcher]] права на запуск 
	* нужно ли разрешать [[#Intent]] (если не явный)
2. **AMS** решает в каком процессе запускать [[#Activity]]: если нет существующего процесса, то создается новый
3. Создание процесса
	* **AMS**  отправляет сокет запрос процессу [[#Zygote]]
	* Zygote получает запрос и вызвает метод `fork()`, который копирует ее содержимое в новый процесс с **Android Frameworks**
	* в этом процессе вызывает метод `main()` класс `android.app.ActivityThread` (**UI thread**)

**Final**
1. `ActivityThread.main()` - точка входа в процесс приложения.
	* инициализируется `Looper`
	* создается `ApplicationThread` это объект `Binder`, который является мостом между **AMS**  и приложением, через него приложение получает команды от системы (запустить `Activity`, остановить итд)
2. **AMS**  получив регистрацию отправляет команду `bindApplication()` через `ApplicaitonThread`
```java
// Упрощенная схема ActivityThread.main()
public static void main(String[] args) {
    ...
    // 1. Инициализация главного Looper
    Looper.prepareMainLooper();
    ...
    // 2. Создание ЭКЗЕМПЛЯРА ActivityThread (синглтон для процесса)
    ActivityThread thread = new ActivityThread();
    
    // 3. !!! Важнейший шаг: Привязка к AMS и создание ApplicationThread !!!
    thread.attach(false, startSeq);
    ...
    // 4. Запуск цикла сообщений главного потока
    Looper.loop();
    ...
}
// Упрощенная схема ActivityThread.attach()
private void attach(boolean system, long startSeq) {
    ...
    // 1. Создается экземпляр ApplicationThread.
    // Он является Binder-объектом (наследник Binder), который может принимать
    // удаленные вызовы.
    final IApplicationThread appThread = new ApplicationThread(this);
    
    // 2. Этот appThread (IApplicationThread) передается в AMS через Binder-вызов.
    // Процесс как бы "звонит" в AMS и говорит: "Я родился, вот мой телефон (appThread),
    // чтобы ты мог мне отдавать команды".
    ActivityManagerService.attachApplication(
        mAppThread /* IApplicationThread */, startSeq);
    ...
}
```

* Этот метод вызывается **НЕ** в главном потоке, а в **Binder-потоке**
* так как нельзя трогать UI и создавать компоненты не в главном потоке, используется `Handler` для передачи работы
```java
// Упрощенная схема ApplicationThread.bindApplication()
public final void bindApplication(...) {
    ...
    // Создается сообщение для главного потока
    AppBindData data = new AppBindData();
    ... // заполнение data
    // Отправка сообщения в главный поток (в очередь Looper'а)
    sendMessage(H.BIND_APPLICATION, data);
}
```
* главный поток обрабатывает `H.BIND_APPLICATION` и вызывает метод `ActivityThread.handleBindApplication()`
```java
// 1. Создается ClassLoader для загрузки классов ИМЕННО ВАШЕГО приложения.
// Он знает, где искать ваш APK.
java.lang.ClassLoader cl = getClassLoader();

// 2. Создается экземпляр вашего Application класса (например, MyApp)
// Это точка, где ваш кастомный код начинает жить!
Application app = mInstrumentation.newApplication(cl, appClass, appContext);

// 3. Ваш Application привязывается к контексту
appContext.setOuterContext(app);
app.attach(appContext);

// 4. ВЫЗЫВАЕТСЯ YOUR Application.onCreate() !!!
mInstrumentation.callApplicationOnCreate(app);
```

## Compilation Steps

### Bytecode JVM (.class)
**Instruments** - `kotlinc`, `javac`

### Annotation processing
**Instruments** 
- old `kapt` - Kotlin Annotation Processing Tool
- new `KSP` - Kotlin Symbol Processing
	в 2 раза быстрее чем `kapt`, так как работает напрямую с Kotlin AST (PST) (с синтаксическим древом после frontend компиляции)

### Dex compilation (.class -> .dex)
**Instruments**: `D8`, `R8` (для минификации)
`R8` - is D8 but integrated with obfuscation and optimization (like proguard) 
преобразовывает байт-код JVM, рассчитанный на стековую виртуальную машину, в формат Dalvik Executable (DEX), оптимизированный для регистровой виртуальной машины **Dalvik/ART**.

### Compilation resources and Manifest
**Instruments**: `AAPT2` (Android Asset Packaging Tool 2)

### Compilation to native code (AOT/JIT) *on client side*
**Instrument**: `dex2oat`
`ART` (Android Runtime) - среда выполнения, а `dex2oat` компилятор
>[!info]
> Раньше использовался DVM(Dalvik), метод компиляции JIT (Just In Time) `dexopt`
> Начиная с Android 5.0, используется ART, метод компиляции AOT(Ahead Of Time) `dex2oat`
> Начиная с Android 7, используется микс JIT и AOT

Использьуется 2 вида компиляции:
* **AOT** (Ahead Of Time) - во время установки компилирует в нативный код и сохраняет
	* плюсы - быстрый запуск тк код нативный 
	* минусы - долгая установка и занимает много памяти
* **JIT** (Just In Time) - при первом запуске приложение выполняется в интерпретаторе `ART`, **JIT** компилирует часто используемые методы и кэширует их, `ART` собирает профиль того какие методы чаще всего вызываются и когда телефон находится без действии(+на зарядке) `dex2oat` может запустить **AOT** и закешировать эти методы

## Startup Components

When the system starts a component, it starts the process for that app, if it’s not already running and instantiates the classes needed for that component. So if app starts the activity in the camera app, that activity runs in the process that belongs to the camera app.

Because the system runs each app in separate process, your app cant directly activate a component from another app. However, the android system can. To activate component from other app, you send message to the system that specifies your intent to start particular component.

**Activating components**

There are separate methods for activating each type of component:

- You can start an activity or give it something new to do by passing an `Intent` to [startActivity()](https://developer.android.com/reference/android/content/Context#startActivity(android.content.Intent)) or, when you want the activity to return a result, [`startActivityForResult()`](https://developer.android.com/reference/android/app/Activity#startActivityForResult(android.content.Intent,%20int)).
- On Android 5.0 (API level 21) and higher, you can use the [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler) class to schedule actions. For earlier Android versions, you can start a service or give new instructions to an ongoing service by passing an `Intent` to [`startService()`](https://developer.android.com/reference/android/content/Context#startService(android.content.Intent)). You can bind to the service by passing an `Intent` to [`bindService()`](https://developer.android.com/reference/android/content/Context#bindService(android.content.Intent,%20android.content.ServiceConnection,%20int)).
- You can initiate a broadcast by passing an `Intent` to methods such as [`sendBroadcast()`](https://developer.android.com/reference/android/content/Context#sendBroadcast(android.content.Intent)) or [`sendOrderedBroadcast()`](https://developer.android.com/reference/android/content/Context#sendOrderedBroadcast(android.content.Intent,%20java.lang.String)).
- You can perform a query to a content provider by calling [query()](https://developer.android.com/reference/android/content/ContentProvider#query(android.net.Uri,%20java.lang.String[],%20android.os.Bundle,%20android.os.CancellationSignal)) on a `ContentResolver`.

## Manifest File

Manifest file needed for
- declare components
- user permission e.g. internet access, read-access to  user’s contact
- hardware or software features
- …

### Declaring components

The primary task of the manifest is to inform the system about components. 

Activities, services and content providers that added to the app but not declared in the manifest file, aren’t visible to the system and can never run. However, broadcast receivers can either be declared in the manifest or created dynamically in code as [`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver)objects and registered with the system by calling [`registerReceiver()`](https://developer.android.com/reference/android/content/Context#registerReceiver(android.content.BroadcastReceiver,%20android.content.IntentFilter)).

### Declare app requirements

There are a variety of devices powered by Android, and not all of them provide the same features and capabilities. To prevent your app from being installed on devices that lack features needed by your app, it's important that you clearly define a profile for the types of devices your app supports by declaring device and software requirements in your manifest file.

Most of these declarations are informational only. The system doesn't read them, but external services such as Google Play do read them to provide filtering for users when they search for apps from their device.

For example, suppose your app requires a camera and uses APIs introduced in Android 8.0 (API level 26). You must declare these requirements. 

### Intent

*intent* - is a messaging object you can use to request an action from other app component.

*intent-filter* - is an expression in app’s manifest file that specifies the type of intents that the component would like to receive. For example, by declaring intent filter for an activity, other apps can use this activity. If it not declared then it can start only with explicit intent. For example: declaring code below, can be used as an email app, and will be launched or purposed as an email sender by ACTION_SEND past to startActivity()

```java
<intent-filter>
<action android:name="android.intent.action.SEND" />
<data android:type="*/*"/>
</intent-filter>
```

## SDK versions and Android software development toolkit

SDK - Software development kit is set of tools, libs, examples, api, sources of code (or implementation of api).

**minSdkVerions** is minimal API version of the app that can run on system. It sets to 1 by default.

**compileSdkVersion.** It is used only for a gradle indication in what SDK version to app compile. It helps developers access to all API available up to level the API set to compileSdkVersion.

**targetSdkVersion -** This value indicates the API level at which the application was developed. Don’t be confused with compileSdkVersion. Compile used only in compile time and make new API’s available for developers. Target is part of APK and changes runtime behaviour.

Examples that app can acts differently when sdk version of a device doesn’t same as targetSdkVersion:

1. Permissions Behavior: Starting from Android 6.0 (API level 23), the runtime permission model was introduced. If an app's targetSdkVersion is set to 22 or lower, it won't request permissions at runtime. However, if the targetSdkVersion is set to 23 or higher, the app will need to request permissions at runtime from the user.
2. Adaptive Icons: Android Oreo (API level 26) introduced adaptive icons that can display in different shapes on different devices. Apps targeting this API level or higher can utilize adaptive icons, allowing them to provide a more consistent and visually appealing experience across various devices.
3. Background Execution Limits: Android Oreo (API level 26) introduced background execution limits to improve battery life. Apps targeting this API level or higher may have restrictions imposed on background services, broadcasts, and location updates.
4. Notification Channels: Android Oreo (API level 26) introduced notification channels, allowing users to have more control over the types of notifications they receive from an app. Apps targeting this API level or higher can make use of notification channels to categorize and prioritize notifications.
5. Security Enhancements: Each new Android version introduces security enhancements. By targeting a higher targetSdkVersion, your app can take advantage of the latest security features and improvements available in that version.

https://stackoverflow.com/questions/76262834/how-does-my-app-work-when-targetsdk-is-higher-than-compile-sdk

https://medium.com/androiddevelopers/picking-your-compilesdkversion-minsdkversion-targetsdkversion-a098a0341ebd

## ANR (Application Not Responding) → [https://developer.android.com/topic/performance/vitals/anr](https://developer.android.com/topic/performance/vitals/anr)

ANR is a Application Not Responding error that occurs when main thread is blocked for too long. When app in foreground, dialog will be displayed.

- **Input dispatching timed out:** If your app has not responded to an input
event (such as key press or screen touch) within 5 seconds.
- **Executing service:** If a service declared by your app cannot finish
executing `Service.onCreate()` and
`Service.onStartCommand()`/`Service.onBind()` within a few seconds.
- **Service.startForeground() not called:** If your app uses
`Context.startForegroundService()` to start a new service in the foreground,
but the service then does not call `startForeground()` within 5 seconds.
- **Broadcast of intent:** If a
[`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver)
hasn't finished executing within a set amount of time. If the app has any
activity in the foreground, this timeout is 5 seconds.

Diagnose ANR’s

There are some common patterns to look for when diagnosing ANRs:

- The app is doing slow operations involving I/O on the main thread.
- The app is doing a long calculation on the main thread.
- The main thread is doing a synchronous binder call to another process, and
that other process is taking a long time to return.
- The main thread is blocked waiting for a synchronized block for a long
operation that is happening on another thread.
- The main thread is in a deadlock with another thread, either in your
process or via a binder call. The main thread is not just waiting for a long
operation to finish, but is in a deadlock situation. For more information,
see [Deadlock](https://en.wikipedia.org/wiki/Deadlock) on Wikipedia.

## Looper
В Android главный поток не всегда занят выполнением какой-либо задачи и часто находиться в состоянии ожидания каких-либо действий. Android использует 3 сущности для реализации такого поведения: **Looper, MessageQueue, Handler**

**Looper** запускает цикл обработки сообщений связанный с потоком. Поток работает пока связанный с ним лупер не будет остановлен. Для создания лупера вызывается метод Looper.prepare() для запуска Looper.loop(). Лупер главного потока можно получить через Looper.getMainLooper()

**Handler** обработчик сообщений, который вызывается между методами Looper.prepare() & Looper.loop(). Передает сообщения в MessageQueue

**MessageQueue** очередь сообщений. Через Handler передаются сообщения в очередь. Можно получить очередь текущего потока методом Loop.myQueue()

Для остановки лупера есть 2 метода quit()  & quitSafely() - методы не статические, они вызываются на объекте лупера. Лупер текущего потока можно получить методом Looper.myLooper().

```java
  class LooperThread extends Thread {
      public Handler mHandler;
	
      public void run() {
          Looper.prepare();
		
          mHandler = new Handler(Looper.myLooper()) {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };
		
          Looper.loop();
      }
  }
```

## View
### Constructors
* `constructor(Context)` - used when we want to create custom view from code, not from xml
* `constructor(Context, AttributeSet)` - used for creating view from xml with attributes that set in xml
* `constructor(Context, AttributeSet, Int(defStyleAttr))` - used with setting style from theme
* `constructor(Context, AttributeSet, Int(defStyleAttr), Int(defStyleRes))`  - used with setting style from theme or/and with resource style
### Methods
* `onAttachToWindow()` - after this method called our view is attached to our’s Activity and know about all elements that in
* `onMeasure(widthMeasureSpec, heightMeasureSpec)` - the point of this method is define size and location our view on screen, it take 2 params which in term represents as measurement requirements the width and height for view. When this method overrode then `setMeasuredDimension(width, height)` needed to call.  
	**Modes**:
	* *EXACTLY*(точно) - размер должен быть указан точно, может быть указан в layout_width с фиксированным значением или match_parent.
	* *AT_MOST*(не больше) - может быть любого размера не превышающего указанный максимальный размер, например layout_height=”wrap_content”
	* *UNSPECIFIED* - размер может быть любым, не ограниченным размером родителя
	```kotlin
		override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
		    val desiredWidth = 100 // Предполагаемая ширина View
		    val desiredHeight = 100 // Предполагаемая высота View
		
		    val widthMode = MeasureSpec.getMode(widthMeasureSpec)
		    val widthSize = MeasureSpec.getSize(widthMeasureSpec)
		    val heightMode = MeasureSpec.getMode(heightMeasureSpec)
		    val heightSize = MeasureSpec.getSize(heightMeasureSpec)
		
		    val width = when (widthMode) {
		        MeasureSpec.EXACTLY -> widthSize // Задан конкретный размер для ширины
		        MeasureSpec.AT_MOST -> min(desiredWidth, widthSize) // Размер не должен превышать заданный размер
		        else -> desiredWidth // Задать предпочтительный размер, если точного или максимального размера не задано
		    }
		
		    val height = when (heightMode) {
		        MeasureSpec.EXACTLY -> heightSize // Задан конкретный размер для высоты
		        MeasureSpec.AT_MOST -> min(desiredHeight, heightSize) // Размер не должен превышать заданный размер
		        else -> desiredHeight // Задать предпочтительный размер, если точного или максимального размера не задано
		    }
		
		    setMeasuredDimension(width, height) // Устанавливаем фактический размер View
		}
	```
* `onLayout()` - called on every change in size and position View. Typically this method override in custom view only when it has child view’s, which need to be placed in certain order.
* `onDraw()` ****- is main method to work with custom view. it is used Canvas(2-D холст) object to draw graphic elements. Paint and path for style and form of elements. This method may cause to large memory consumption, may cause load on the garbage collector, so don’t create objects in this method.
* `onSizeChanged(w: Int, h: Int, oldW: Int, oldH: Int)` - called when screen changed orientation or parent view size changed
* `onSaveInstanceState()`, `onRestoreInstanceState(Parcelable)` - TODO
* `onTouchEvent(MotionEvent)` - если вернет true, если мы обработали event, если false, то произойдет вызовуться `onTouchEvent` у родителей View
	![Screenshot from 2024-05-05 19-26-38.png|600](Screenshot_from_2024-05-05_19-26-38.png)
* `dispatchTouchEvent(MotionEvent)` - если вернет true, то тачи будут обрабаатываться, если false, то нет.
* `onInterceptTouchEvent()` (только для ViewGroup) - если возвращает false, то ивент пробрасывается вниз по иеррархии, если true, то ViewGroup обрабатывает их сама, далее onTouchEvent.
* `requestLayout()` & `invalidate()` - нужно понимать, что эти методы не вызывают моментальную перерисовку, а скорее устанавливают флаги, которые гласят, что на следующем тике лупера их нужно перерисовать, поэтому при многочиленном вызове этих методов, не означает, что методы жизненного цикла будут вызваны несколько раз. Например: `invalidate()` вызван 5 раз, но `onDraw()` вызовится 1 раз.
![view_lifecycle.png|300](view_lifecycle.png)

## RecyclerView, Adapter

### DiffUtil
Instrument for adapter optimization 
#### Methods:
* `areItemsTheSame(oldItem, newItem)` - method that checks if objects are same (for example check ids)
* `areContentsTheSame(oldItem, newItem)` - method that checks if content is changed (if previous method returned true, this method will be called)
* `getChangePayload(oldItem, newItem): Object` - when previous method returned false, then this method will be called to get payload about the change. We can return some particular object, which stands for some changes about field in holder

## Layouts
* FrameLayout
* RelativeLayout
* LinearLayout
* ConstraintLayout

## Jetpack Compose (TODO: add more info)

### Recomposition basics
* recompositions skips as many composable functions and lambdas as possible
- recompositions may cancelled
- composable functions may executes in parallel
- composable functions may executes in any order
- composable functions may executes frequently as much as every frame of animation


### Lifecycle
Enters composition -> Repcomposes N times -> Leave  composition
* композиция -
* макет
* отрисовка
* уничтожение
![[lifecycle-composition.png|600]]
Recomposition triggers by `State<T>` . Compose tracks these and triggers all composables that use these states and also composables that cannot be skipped. 

State hoisting - https://developer.android.com/develop/ui/compose/state-hoisting

#####  Проверить
* если передавать Unit через SharedFlow, то будет ли реакция на recompose? (наверно нет)
* mutex

# Kotlin | Java
---
## Language
### Object, Any
Object is a root parent class in java that all classes are inherit
#### methods:
* equals - 
* hashCode
* toString
* finalize - it called by garbage collector when it decides to get rid of an object
* getClass - returns type of an object `Class<Object>`
* notify -
* notifyAll
* wait - stops a thread 

### Nothing
Is a type that inherits all types of object

example of usage - [https://medium.com/@ultimate_guy/a-simple-real-world-example-of-nothing-in-kotlin-33f5d7d01fa6](https://medium.com/@ultimate_guy/a-simple-real-world-example-of-nothing-in-kotlin-33f5d7d01fa6)

It can be used for functions that will throw exceptions or infinite loops: 
```kotlin
fun example(): Nothing = throw Exception() or fun example(): Nothing =  while(true) {}
```
## Garbage collector



# Coroutines
---
**Coroutine** - экземпляр приостанавливаемого вычисления, концептуально схож с потоком тем, что он берет блок кода и выполняет его одновременно с другим кодом. Однако корутина не привязана к потоку и может приостановиться и продолжить свое выполнение в другом потоке 

**StructuredConcurrency** - это принцип, который означает, что корутины могут быть запущены в определенной области(CoroutineScope), что ограничивает жизнь корутины. Он гарантирует, что корутины не потеряются и не протекут.

**CoroutineScope** - можносоздавать свои corutineScope билдеры в которых будут запускаться корутины и он не заверается пока все корутины внутри не закончат выполнение своего кода.

**CoroutineBuilders**

**CoroutineContext**

**Channels**

**Flow - flow builder** 

## Multithreading




# OTHER???
---
- JAVA
    - Collections - как под капотом работают - в чем разница array, arraylist
    - types refereces (типы ссылок)!!!
    - synchronized способы синхронизации

- Android
    - apk - what inside
    - components - all
    - customview - ondraw, onmeassure vs onlayout, all methods

- rxjava
    - cold vs hot data

- dagger
    - scope - для чего, что это?
    - subcomponent - слабость?
    - 


# Additional materials
---
https://habr.com/ru/companies/tinkoff/articles/703548/ - как работают Activity

[Q/A](QA.md)

[Вопросы для работадателя](Вопросы%20для%20работадателя.md)

[Algorithms](Algorithms.md)

[Testing](Testing.md)


