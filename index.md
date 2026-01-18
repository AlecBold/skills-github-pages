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

