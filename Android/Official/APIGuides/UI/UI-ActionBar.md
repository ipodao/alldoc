Action bar有三大功能：提示用户当前在哪，提供用户功能入口，提供导航、视图切换功能（Tab或下拉列表）。

![](actionbar@2x.png)

Figure 1. An action bar that includes the [1] app icon, [2] two action items, and [3] action overflow.

Action Bar提供以下功能：

参见[Action Bar设计指导](http://developer.android.com/design/patterns/actionbar.html)。

ActionBar APIs是Android 3.0 (API level 11)添加的，可以通过支持库可以兼容到Android 2.1 (API level 7)。本文档使用支持库做介绍。

## 添加Action Bar

创建活动，继承 [ActionBarActivity](http://developer.android.com/reference/android/support/v7/app/ActionBarActivity.html)。

使用（或扩展）某个 [Theme.AppCompat](http://developer.android.com/reference/android/support/v7/appcompat/R.style.html#Theme_AppCompat) 主题。例如：

	<activity android:theme="@style/Theme.AppCompat.Light" ... >

> **On API level 11 or higher**  任何使用 [Theme.Holo](http://developer.android.com/reference/android/R.style.html#Theme_Holo) 主题的Activity都带Action Bar。如果`targetSdkVersion`或`minSdkVersion`大于11，Theme.Holo是默认主题。如果不想要Action Bar，设置主题为 [Theme.Holo.NoActionBar](http://developer.android.com/reference/android/R.style.html#Theme_Holo_NoActionBar)。

### 移除Action Bar

在运行时可以调用`hide()`隐藏Action Bar。

	ActionBar actionBar = getSupportActionBar();
	actionBar.hide();

> **On API level 11 or higher** Get the ActionBar with the [getActionBar()](http://developer.android.com/reference/android/app/Activity.html#getActionBar()) method.

When the action bar hides, the system adjusts your layout to fill the screen space now available. 调用`show()`可以再让Action Bar回来。

隐藏或移除Action Bar将导致活动重新布局。如果活动能够需要经常显示、隐藏Action Bar，考虑使用遮盖（overlay）模式。遮盖模式下，Action Bar在Activity布局之上，占据顶部空。于是隐藏显示Action Bar不影响活动布局。要启用遮盖模式，为活动自定义一个主题，设置` windowActionBarOverlay`为true。

### 不使用icon，使用logo

模式在Action Bar上使用应用图标（`<application>`或`<activity>`元素的`icon`特性）。但如果指定了`logo`格形，Action Bar将使用logo。

## 添加Action项

The action bar provides users access to the most important action items **relating to the app's current context**. 直接显示在action bar上的图标或位子称为action按钮。不重要的或放不下的隐藏在action overflow。按下overflow按钮可以显示overflow。

![](actionbar-item-withtext.png)

Figure 2. Action bar with three action buttons and the overflow button.

启动活动时，系统调用活动的`onCreateOptionsMenu()`，重启Menu，定义action items。

res/menu/main_activity_actions.xml

	<menu xmlns:android="http://schemas.android.com/apk/res/android" >
	    <item android:id="@+id/action_search"
	          android:icon="@drawable/ic_action_search"
	          android:title="@string/action_search"/>
	    <item android:id="@+id/action_compose"
	          android:icon="@drawable/ic_action_compose"
	          android:title="@string/action_compose" />
	</menu>

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
	    // Inflate the menu items for use in the action bar
	    MenuInflater inflater = getMenuInflater();
	    inflater.inflate(R.menu.main_activity_actions, menu);
	    return super.onCreateOptionsMenu(menu);
	}

要菜单项直接显示在Action Bar上，设置特性`showAsAction="ifRoom"`：

	<menu xmlns:android="http://schemas.android.com/apk/res/android"
	      xmlns:yourapp="http://schemas.android.com/apk/res-auto" >
	    <item android:id="@+id/action_search"
	          android:icon="@drawable/ic_action_search"
	          android:title="@string/action_search"
	          yourapp:showAsAction="ifRoom"  />
	    ...
	</menu>

If there's not enough room for the item in the action bar, it will appear in the action overflow.

> **使用支持库中的XML特性** 注意到`showAsAction`特性使用的命名空间。当XML特性来自支持库时，需要使用自定义命名空间。因为老设备的Android框架没有该特性。

若同时定义了`title`和`icon`特性，默认，Action项只会显示图标。如果要显示标题文本：

	<item yourapp:showAsAction="ifRoom|withText" ... />

> Note: The "withText" value is a hint to the action bar that the text title should appear. The action bar will show the title when possible, but might not if an icon is available and the action bar is constrained for space.

即使多数时候不显示，还是要定义title。如果Action Item隐藏到了Overflow中，则只会显示文本。If the action item appears with only the icon, a user can long-press the item to reveal a tool-tip that displays the action title.

The icon is optional, but recommended. For icon design recommendations, see the [Iconography](http://developer.android.com/design/style/iconography.html#action-bar) design guide. You can also download a set of standard action bar icons (such as for Search or Discard) from the [Downloads](http://developer.android.com/design/downloads/index.html) page.

You can also use "always" to declare that an item always appear as an action button. However, you should not force an item to appear in the action bar this way. Doing so can create layout problems on devices with a narrow screen. It's best to instead use "ifRoom" to request that an item appear in the action bar, but allow the system to move it into the overflow when there's not enough room. However, it might be necessary to use this value if the item includes an action view that cannot be collapsed and must always be visible to provide access to a critical feature.

### 处理点击

按下一个Action，系统调用活动的`onOptionsItemSelected()`方法。传入选中的`MenuItem`。识别此项可以调用`getItemId()`，返回菜单项的`android:id`。

	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
	    // Handle presses on the action bar items
	    switch (item.getItemId()) {
	        case R.id.action_search:
	            openSearch();
	            return true;
	        case R.id.action_compose:
	            composeMessage();
	            return true;
	        default:
	            return super.onOptionsItemSelected(item);
	    }
	}

> Note: 如果从Fragment充气菜单项（通过Fragment的`onCreateOptionsMenu()`，系统调用Fragment的`onOptionsItemSelected()`。However, the activity gets a chance to handle the event first, so the system first calls onOptionsItemSelected() on the activity, before calling the same callback for the fragment. To ensure that any fragments in the activity also have a chance to handle the callback, always pass the call to the superclass as the default behavior instead of returning false when you do not handle the item.

### Using split action bar

当活动运行在窄屏上时，Split action bar在屏幕底部显示一个独立的Bar，显示所有的Action Item。

To enable split action bar when using the support library, you must do two things:

1. 在每个`<activity>`或`<application>`上添加`uiOptions="splitActionBarWhenNarrow"`。This attribute is understood only by API level 14 and higher (it is ignored by older versions).
1. To support older versions, add a <meta-data> element as a child of each <activity> element that declares the same value for `android.support.UI_OPTIONS`.

例子：

	<manifest ...>
	    <activity uiOptions="splitActionBarWhenNarrow" ... >
	        <meta-data android:name="android.support.UI_OPTIONS"
	                   android:value="splitActionBarWhenNarrow" />
	    </activity>
	</manifest>

Using split action bar also allows [navigation tabs](http://developer.android.com/guide/topics/ui/actionbar.html#Tabs) to collapse into the main action bar if you remove the icon and title (as shown on the right in figure 3). To create this effect, disable the action bar icon and title with setDisplayShowHomeEnabled(false) and setDisplayShowTitleEnabled(false).

![](actionbar-splitaction@2x.png)

Figure 3. Mock-ups showing an action bar with tabs (left), then with split action bar (middle); and with the app icon and title disabled (right).

## Navigating Up with the App Icon







