## 运行时变化

设备配置在运行时改变后，Android重启运行中的Activity（调用`onDestroy()`和`onCreate()`）。重启是为了重新加载适应配置的替换资源。

在销毁活动前Android会调用`onSaveInstanceState()`，在该方法中可以保存应用的状态。可以在`onCreate()`或`onRestoreInstanceState()`中恢复状态。

To learn how you can restore your activity state, read about the [Activity lifecycle](http://developer.android.com/guide/components/activities.html#Lifecycle).

However, you might encounter a situation in which restarting your application and restoring significant amounts of data can be costly and create a poor user experience. In such a situation, you have two other options:

- 利用Fragment驻留状态
- 不要让系统因配置改变而重启Activity。实现回调，自己监听改变，手工更新活动。

### 利用Fragment驻留状态

如果重启活动需要恢复大量数据，重新建立网络连接，或进行其他量大的操作，则因为配置改变而重启可能减慢用户体验。且利用`onSaveInstanceState()`保存的`Bundle`不一定能恢复Activity所有状态，`Bundle`不适合携带大对象（如bitmaps）和需要序列化的对象（会消耗大量内存，导致配置改变变慢）。一个解决办法是，保留一个`Fragment`，用来重新初始化你的活动。该Fragment可以引用你想驻留的状态对象。

当系统因配置改变关闭你的活动时，标记为驻留（retain）的Fragment不会被销毁。

> 注意！尽管可以保存任何对象，但不要保存与Activity有引用关系的对象，如`Drawable`、`Adapter`、`View`等，**这些对象均与`Context`关联**。If you do, it will leak all the views and resources of the original activity instance. (Leaking resources means that your application maintains a hold on them and they cannot be garbage-collected, 因此大量内存被消耗。)

过程：

1. Extend the Fragment class and declare references to your stateful objects.
1. Call setRetainInstance(boolean) when the fragment is created.
1. Add the fragment to your activity.
1. Use FragmentManager to retrieve the fragment when the activity is restarted.

例子：

	public class RetainedFragment extends Fragment {
	
	    // data object we want to retain
	    private MyDataObject data;
	
	    // this method is only called once for this fragment
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        // retain this fragment
	        setRetainInstance(true);
	    }
	
	    public void setData(MyDataObject data) {
	        this.data = data;
	    }
	
	    public MyDataObject getData() {
	        return data;
	    }
	}

	public class MyActivity extends Activity {
	
	    private RetainedFragment dataFragment;
	
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.main);
	
	        // find the retained fragment on activity restarts
	        FragmentManager fm = getFragmentManager();
	        dataFragment = (DataFragment) fm.findFragmentByTag(“data”);
	
	        // create the fragment and data the first time
	        if (dataFragment == null) {
	            // add the fragment
	            dataFragment = new DataFragment();
	            fm.beginTransaction().add(dataFragment, “data”).commit();
	            // load the data from the web
	            dataFragment.setData(loadMyData());
	        }
	
	        // 直接调用Fragment的public方法访问，dataFragment.getData()
	        ...
	    }
	
	    @Override
	    public void onDestroy() {
	        super.onDestroy();
	        // store the data in the fragment
			// {{Fragment中的数据不用实时更新，销毁前更新一次就好！}}
	        dataFragment.setData(collectMyLoadedData());
	    }
	}

### 自己处理配置改变

If your application doesn't need to update resources during a specific configuration change and you have a performance limitation that requires you to avoid the activity restart, then you can declare that your activity handles the configuration change itself, which prevents the system from restarting your activity.

> Note: Handling the configuration change yourself can make it much more difficult to use alternative resources, because the system does not automatically apply them for you. This technique should be considered a last resort when you must avoid restarts due to a configuration change and is not recommended for most applications.

To declare that your activity handles a configuration change, edit the appropriate `<activity>` element in your manifest file to include the `android:configChanges `attribute with a value that represents the configuration you want to handle. Possible values are listed in the documentation for the `android:configChanges` attribute (the most commonly used values are "`orientation`" to prevent restarts when the screen orientation changes and "`keyboardHidden`" to prevent restarts when the keyboard availability changes). You can declare multiple configuration values in the attribute by separating them with a pipe `|` character.

For example, the following manifest code declares an activity that handles both the screen orientation change and keyboard availability change:

	<activity android:name=".MyActivity"
	          android:configChanges="orientation|keyboardHidden"
	          android:label="@string/app_name">

Now, when one of these configurations change, MyActivity does not restart. Instead, the MyActivity receives a call to `onConfigurationChanged()`. This method is passed a `Configuration` object that specifies the new device configuration. By reading fields in the Configuration, you can determine the new configuration and make appropriate changes by updating the resources used in your interface. At the time this method is called, your activity's Resources object is updated to return resources based on the new configuration, so you can easily reset elements of your UI without the system restarting your activity.

> Caution: Beginning with Android 3.2 (API level 13), the "screen size" also changes when the device switches between portrait and landscape orientation. Thus, if you want to prevent runtime restarts due to orientation change when developing for API level 13 or higher (as declared by the minSdkVersion and targetSdkVersion attributes), you must include the "`screenSize`" value in addition to the "`orientation`" value. That is, you must decalare `android:configChanges="orientation|screenSize"`. However, if your application targets API level 12 or lower, then your activity always handles this configuration change itself (this configuration change does not restart your activity, even when running on an Android 3.2 or higher device).

For example, the following onConfigurationChanged() implementation checks the current device orientation:

	@Override
	public void onConfigurationChanged(Configuration newConfig) {
	    super.onConfigurationChanged(newConfig);
	
	    // Checks the orientation of the screen
	    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
	        Toast.makeText(this, "landscape", Toast.LENGTH_SHORT).show();
	    } else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT){
	        Toast.makeText(this, "portrait", Toast.LENGTH_SHORT).show();
	    }
	}

The Configuration object represents all of the current configurations, not just the ones that have changed. Most of the time, you won't care exactly how the configuration has changed and can simply re-assign all your resources that provide alternatives to the configuration that you're handling. For example, because the Resources object is now updated, you can reset any ImageViews with setImageResource() and the appropriate resource for the new configuration is used (as described in Providing Resources).

Notice that the values from the Configuration fields are integers that are matched to specific constants from the Configuration class. For documentation about which constants to use with each field, refer to the appropriate field in the Configuration reference.

> Remember: When you declare your activity to handle a configuration change, you are responsible for resetting any elements for which you provide alternatives. If you declare your activity to handle the orientation change and have images that should change between landscape and portrait, you must re-assign each resource to each element during onConfigurationChanged().

If you don't need to update your application based on these configuration changes, you can instead not implement onConfigurationChanged(). In which case, all of the resources used before the configuration change are still used and you've only avoided the restart of your activity. However, your application should always be able to shutdown and restart with its previous state intact, so you should not consider this technique an escape from retaining your state during normal activity lifecycle. Not only because there are other configuration changes that you cannot prevent from restarting your application, but also because you should handle events such as when the user leaves your application and it gets destroyed before the user returns to it.

For more about which configuration changes you can handle in your activity, see the [android:configChanges](http://developer.android.com/guide/topics/manifest/activity-element.html#config) documentation and the [Configuration](http://developer.android.com/reference/android/content/res/Configuration.html) class.