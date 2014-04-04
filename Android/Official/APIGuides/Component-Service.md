# Service

[来源](http://developer.android.com/guide/components/services.html)

SAMPLES

* [ServiceStartArguments](http://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/ServiceStartArguments.html)
* [LocalService](http://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/LocalService.html)

组件可以绑定到一个服务，与之交互，甚至可以做跨进程通讯（interprocess communication (IPC)。

服务可以有两种形态：

* Started  
其他应用组件（如活动）通过调用`startService()`令服务启动。启动后，服务可以一直在后台运行，即使启动它的组件被销毁了。服务一般只做一个操作，不返回结果给调用者。例如下载一个文件。完成后，结束自己。
* Bound  
一个应用组件通过`bindService()`绑定到服务。绑定的服务提供client-server接口，允许组件与服务交互：发请求，得到结果，甚至做跨进程通讯(IPC)。只有当组件绑定到服务是服务才运行着。服务可被多个组件绑定。当它们都解除绑定后，服务被销毁。

服务可以既支持启动（一直运行下去）又支持绑定。关键看你如何实现几个回调方法`onStartCommand()`还是`onBind()`。

不管采用启动还是绑定，一个应用（甚至其他独立应用）使用服务的方式，都是利用`Intent`。可以在装箱文件中将服务声明为私有，阻止其他应用访问。

> **Caution**: *服务器运行在宿主进程的主线程*——服务器不是运行在单独的进程（除非你不这 样设定）。如果服务要做一些CPU密集或阻塞的操作，如播放MP3或网络，应该在服务中创建新线>程。防止ANR。


# 基础

> 使用服务还是线程？  
如果想只在用户与应用交互时，做些操作，应在`onCreate()`创建一个线程，在`onStart()`中启动它，在`onStop()`中停止。或者考虑使用`AsyncTask`或`HandlerThread`。参见[Processes and Threading](http://developer.android.com/guide/components/processes-and-threads.html)。

创建服务需要创建`Service`的子类。关键回调方法：

* onStartCommand()  
当活动或其他组件，调用`startService()`请求活动启动时，系统调用此方法。此方法执行后，服务被启动，将在后台无限运行。在这种模式下，需要显式停止服务，可以通过stopSelf()或stopService()方法。如果服务只用于绑定，不需要实现此方法。
* onBind()  
当另外的组件想要绑定到服务时（调用`bindService()`），系统调用该方法。此方法需要返回一个`IBinder`。该方法必须实现，但如果不想被绑定，只需要返回null。
* onCreate()  
当活动被第一次创建时，系统调用该方法。（该方法在`onStartCommand()`或`onBind()`）之前执行。如果服务已在运行，不会调用该方法。
* onDestroy()  
当服务不再被使用，要被摧毁时，系统调用该方法。服务利用该方法清理资源，如线程、监听器、receivers。该方法是服务收到的最后一次调用。

系统杀死服务后，若资源又可用了，系统会尽快重启服务（虽然这取决于`onStartCommand()`方法的返回值，后面讨论）。因此，代码要优雅的处理重启。

### 在装箱文件中声明服务

服务必须在装箱文件中声明。

	<manifest ... >
	  ...
	  <application ... >
	      <service android:name=".ExampleService" />
	      ...
	  </application>
	</manifest>

`android:name`是唯一必须指定的特性。发布应用后，这个名字就不能改了，because if you do, you risk breaking code due to dependence on explicit intents to start or bind the service （参见 [Things That Cannot Change](http://android-developers.blogspot.com/2011/06/things-that-cannot-change.html)）。

为了保证应用安全，总是显式的启动或绑定服务，不要为服务声明intent filters。If it's critical that you allow for some amount of ambiguity as to which service starts, you can supply intent filters for your services and exclude the component name from the Intent, but you then must set the package for the intent with `setPackage()`, which provides sufficient disambiguation for the target service.

若想限制只有你的App可以使用此服务，设置`android:exported`特性为`false`。浙江阻止其他App启动你的服务，使用显式Intent也不行。

## 创建Started服务

服务被启动后，它的生命周期独立于启动它的组件，可以在后台无限运行，即使启动它的组件已被销毁。｛｛但进程未退出｝｝。服务可以调用`stopSelf()`停止。或其他组件可以通过`stopService()`停止它。

应用组件调用`startService()`时传入Intent指定要启动的服务，指定数据。服务可以在` onStartCommand()`接收到这个Intent。

> 注意：服务默认运行在应用所在进程的主线程。

要创建一个started方法，可以继承两个类：

* Service：所有服务的基类。使用该类时，记得开新线程处理费时操作。
* IntentService：`Service`的子类。使用后台线程处理所有start请求，一次一个。如果你不需要你的服务并行处理请求，这是最好的选择。你只要实现`onHandleIntent()`，处理start请求的intent。`onHandleIntent()`在默认的后台线程执行。

### 扩展IntentService类

The IntentService does the following:

* 创建一个默认的后台线程，执行所有传给`onStartCommand()`的Intent。
* 创建一个工作队列，一次向你实现的`onHandleIntent()`传一个Intent。
* 在所有start请求被处理后，停止服务，于是你不必再调用`stopSelf()`。
* 实现`onBind()`并返回null。
* 实现`onStartCommand()`，将Intent放入队列。

例子：

	public class HelloIntentService extends IntentService {
	
	  /**
	   * 必须有构造器。调用父类。传入后台线程的名字。
	   */
	  public HelloIntentService() {
	      super("HelloIntentService");
	  }

	  @Override
	  protected void onHandleIntent(Intent intent) {
	      long endTime = System.currentTimeMillis() + 5*1000;
	      while (System.currentTimeMillis() < endTime) {
	          synchronized (this) {
	              try {
	                  wait(endTime - System.currentTimeMillis());
	              } catch (Exception e) {
	              }
	          }
	      }
	  }
	}

若要覆盖回调方法，如`onCreate()`, `onStartCommand()`, `onDestroy()`，记得调用父类的方法。

For example, onStartCommand() must return the default implementation (which is how the intent gets delivered to onHandleIntent()):

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
	    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
	    return super.onStartCommand(intent,flags,startId);
	}

### 扩展`Service`类

下面的类做的实现与`IntentService`一样。That is, for each start request, it uses a worker thread to perform the job and processes only one request at a time.

	public class HelloService extends Service {
	  private Looper mServiceLooper;
	  private ServiceHandler mServiceHandler;
	
	  // Handler that receives messages from the thread
	  private final class ServiceHandler extends Handler {
	      public ServiceHandler(Looper looper) {
	          super(looper);
	      }
	      @Override
	      public void handleMessage(Message msg) {
	          long endTime = System.currentTimeMillis() + 5*1000;
	          while (System.currentTimeMillis() < endTime) {
	              synchronized (this) {
	                  try {
	                      wait(endTime - System.currentTimeMillis());
	                  } catch (Exception e) {
	                  }
	              }
	          }
	          // Stop the service using the startId, so that we don't stop
	          // the service in the middle of handling another job
	          stopSelf(msg.arg1);
	      }
	  }
	
	  @Override
	  public void onCreate() {
	    // 创建并启动线程
	    HandlerThread thread = new HandlerThread("ServiceStartArguments",
	            Process.THREAD_PRIORITY_BACKGROUND);
	    thread.start();
	
	    // Get the HandlerThread's Looper and use it for our Handler
	    mServiceLooper = thread.getLooper();
	    mServiceHandler = new ServiceHandler(mServiceLooper);
	  }
	
	  @Override
	  public int onStartCommand(Intent intent, int flags, int startId) {
	      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
	
	      // For each start request, send a message to start a job and deliver the
	      // start ID so we know which request we're stopping when we finish the job
	      Message msg = mServiceHandler.obtainMessage();
	      msg.arg1 = startId;
	      mServiceHandler.sendMessage(msg);
	
	      // If we get killed, after returning from here, restart
	      return START_STICKY;
	  }
	
	  @Override
	  public IBinder onBind(Intent intent) {
	      // 不支持绑定，于是返回null
	      return null;
	  }
	
	  @Override
	  public void onDestroy() {
	    Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show();
	  }
	}

`onStartCommand()`的返回值表示，服务被杀死后，系统该如何继续服务。返回值取值：

* `START_NOT_STICKY`  
如果系统在`onStartCommand()`返回后杀死了服务，不要重启服务，unless there are pending intents to deliver. This is the safest option to avoid running your service when not necessary and when your application can simply restart any unfinished jobs.
* `START_STICKY`  
如果系统在`onStartCommand()`返回后杀死了服务，重启服务并调用`onStartCommand()`，但不要递送上一个Intent。系统调用`onStartCommand()`，Intent传null，unless there were pending intents to start the service, in which case, those intents are delivered。This is suitable for media players (or similar services) that are not executing commands, but running indefinitely and waiting for a job.
* `START_REDELIVER_INTENT`  
如果系统在`onStartCommand()`返回后杀死了服务，重启服务并调用`onStartCommand()`，递送之前已递送过的最后一个Intent。Any pending intents are delivered in turn. This is suitable for services that are actively performing a job that should be immediately resumed, such as downloading a file.

### 启动服务

例子：

	Intent intent = new Intent(this, HelloService.class);
	startService(intent);

`startService()`方法会立即返回，Android调用服务的`onStartCommand()`方法。如果服务尚未运行，服务会先调用`onCreate()`，再调用`onStartCommand()`。

如果向服务响应一个结果，客户端启动服务时可以创建一个`PendingIntent`做广播（with getBroadcast()），将此`PendingIntent`递送给服务。服务可以用广播返回结果。

多次请求启动服务导致多次调用服务的`onStartCommand()`方法。但关闭服务只需一次（`stopSelf()`或`stopService()`）。

### 停止服务

A started service must manage its own lifecycle. That is, the system does not stop or destroy the service unless it must recover system memory and the service continues to run after onStartCommand() returns. So, the service must stop itself by calling stopSelf() or another component can stop it by calling stopService().

Once requested to stop with stopSelf() or stopService(), the system destroys the service as soon as possible.

但如果服务并发收到多个`onStartCommand()`请求，在你完成请求后，不应该停止服务，因为你可能已经收到一个新的start请求，停止将终止第二个请求。为避免该问题，可以利用`stopSelf(int)` to ensure that your request to stop the service is always based on the most recent start request. That is, when you call stopSelf(int), you pass the ID of the start request (the startId delivered to onStartCommand()) to which your stop request corresponds. Then if the service received a new start request before you were able to call stopSelf(int), then the ID will not match and the service will not stop.

> 注意：为了避免浪费系统资源，耗费电池，完成功能后应该停止服务。If necessary, other components can stop the service by calling stopService(). Even if you enable binding for the service, you must always stop the service yourself if it ever received a call to onStartCommand().

## 创建绑定服务

一个bound服务允许组件通过调用`bindService()`绑定它，建立长期连接（一般不允许组件通过`startService()`启动它）。

绑定服务可以让应用内其他组件与之交互，或令其他应用访问（通过interprocess communication (IPC)）。

`onBind()`方法必须返回一个`IBinder`，定义通讯接口。

客户端和服务器之间的接口必须实现`IBinder`。一旦客户端收到`IBinder`，可以用它与服务交互。

客户端不再需要服务后，调用`unbindService()`解除绑定。

实现绑定服务的方式较多，也复杂，见后面单独章节。

## 给用户通知

服务可以通过Toast Notifications或Status Bar Notifications通知用户。

## 在前台运行服务

前台服务是用户有知觉的服务，因此最好不要因低内存而杀死。前台服务必须在status bar提供一个通知，在"Ongoing"之下，表示通知不能被移除，除非服务被停止，或被移出前台。

例如，播放音乐的服务应该在前台运行，因为用户对它有知觉。status bar的通知告诉用户当前歌曲，通过它还可以启动一个Activity与播放器交互。

要使服务前台运行，调用`startForeground()`。该方法取两个参数，一个唯一标识通知的整数，一个Notification用于status bar。例如：

    Notification notification = new Notification(R.drawable.icon, getText(R.string.ticker_text),
            System.currentTimeMillis());
    Intent notificationIntent = new Intent(this, ExampleActivity.class);
    PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, 0);
    notification.setLatestEventInfo(this, getText(R.string.notification_title),
            getText(R.string.notification_message), pendingIntent);
    startForeground(ONGOING_NOTIFICATION_ID, notification);

> Caution: The integer ID you give to startForeground() must not be 0.

要把通知从前台移除，调用`stopForeground()`。该方法接受一个布尔，表示是否同时移除状态条通知。该方法不会停止服务。但，如果你直接停止了一个在前台运行的服务，通知也会被移除。

## 管理活动生命周期

活动的生命周期，从创建开始到销毁结束。

启动和绑定服务可以同时进行。例如启动服务放音乐，后来，又绑定它，例如获取当前歌曲信息。此时，`stopService()`或`stopSelf()`不会实际停止服务，直到服务被解除绑定。

下面是活动的生命周期回调：

    public class ExampleService extends Service {
        int mStartMode;       // indicates how to behave if the service is killed
        IBinder mBinder;      // interface for clients that bind
        boolean mAllowRebind; // indicates whether onRebind should be used
    
        @Override
        public void onCreate() {
            // The service is being created
        }
        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            // The service is starting, due to a call to startService()
            return mStartMode;
        }
        @Override
        public IBinder onBind(Intent intent) {
            // A client is binding to the service with bindService()
            return mBinder;
        }
        @Override
        public boolean onUnbind(Intent intent) {
            // All clients have unbound with unbindService()
            return mAllowRebind;
        }
        @Override
        public void onRebind(Intent intent) {
            // A client is binding to the service with bindService(),
            // after onUnbind() has already been called
        }
        @Override
        public void onDestroy() {
            // The service is no longer used and is being destroyed
        }
    }


> Note: 与活动不同的是，你不必调用父类的生命周期回调。

By implementing these methods, you can monitor two nested loops of the service's lifecycle:

* 服务的整个生命周期从`onCreate()`到`onDestroy()`方法返回。初始在`onCreate()`中做，在`onDestroy()`中释放所有资源。例如在`onCreate()`创建线程，在`onDestroy()`中停止。
* 服务的活跃（active）生命周期从`onStartCommand()`或`onBind()`开始。
If the service is started, the active lifetime ends the same time that the entire lifetime ends (the service is still active even after onStartCommand() returns). If the service is bound, the active lifetime ends when onUnbind() returns.

> Note: Although a started service is stopped by a call to either stopSelf() or stopService(), there is not a respective callback for the service (there's no onStop() callback). So, unless the service is bound to a client, the system destroys it when the service is stopped—onDestroy() is the only callback received.

## 绑定服务

[来源](http://developer.android.com/guide/components/bound-services.html)

一个bound服务是C/S接口中的服务器端。

### 基础

A client can bind to the service by calling bindService(). When it does, it must provide an implementation of ServiceConnection, 用于监控连接状态。`bindService()`方法再调用后立即返回，无返回值。Android系统建立客户端与服务器连接后，会调用`ServiceConnection`的`onServiceConnected()`方法，传给客户端一个`IBinder`。

只有在第一次连接时，系统会调用服务的`onBind()`方法，拿到一个`IBinder`。以后其他组件绑定时，返回的是同一个`IBinder`。

### 创建Bound服务

服务提供的`IBinder`，供客户端与服务交互。定义`IBinder`接口有几种方式。

* 继承`Binder`类  
如果服务对你的应用私有，与调用者运行在统一进程，应该扩展`Binder`类，在`onBind()`中返回该类的实例。收到`Binder`实例的客户端可以直接访问其公有方法。
推荐使用该方法。除非服务可能被其他应用使用，或被其他进程访问。
* 利用Messenger  
如果需要跨进程，you can create an interface for the service with a [Messenger](http://developer.android.com/reference/android/os/Messenger.html)。服务定义一个Handler，处理不同类型的Message对象。This Handler is the basis for a Messenger that can then share an IBinder with the client, 其他客户端利用Message对象向服务发送命令。客户端还可以定义自己的Messenger，这样服务就能回送数据。  
这是进行interprocess communication (IPC)最简单的方式。因为`Messenger`将所有请求在单个线程中排队，因此你的服务不需要设计成线程安全的。
* 使用AIDL  
AIDL (Android Interface Definition Language) performs all the work to decompose objects into primitives that the operating system can understand and marshall them across processes to perform IPC. 上一节讲的`Messenger`技术，底层实际是基于`AIDL`的。As mentioned above, the Messenger creates a queue of all the client requests in a single thread, so the service receives requests one at a time. If, however, 如果你想让服务并发处理多个请求，则可以直接使用AIDL。此时你的服务必须能解决多线程和线程安全。  
To use AIDL directly, you must create an .aidl file that defines the programming interface. The Android SDK tools use this file to generate an abstract class that implements the interface and handles IPC, which you can then extend within your service.

> 注意：多数应用不应使用AIDL实现绑定服务，因为它需要多线程处理，导致实现更复杂。


