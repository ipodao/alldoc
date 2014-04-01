可以将一个Fragment看做活动的一个模块化的部分。为了重用Fragment UI，你需要将每个Fragment定义为完全自包含的组件，拥有自己的布局和行为。Fragment有自己的生命周期，接受自己的输入事件。可以在活动运行过程中添加或删除fragment。

Fragment可以不带UI；向活动提供后台功能。

Fragment不需要向装箱文件注册。

Fragment是Android 3.0 Honeycomb (API level 11)才引入的。支持库可以支持到Android 1.6 (API level 4)。如果最低支持的API版本高于11，可以不用Support Library，直接使用系统的Fragment类。

# 创建Fragment

注意：Fragment必须有一个默认无参构造器。当宿主Activity被重建后，系统利用该默认构造器实例化Fragment。忘记提供默认构造器不会不抱错，但会导致不确定行为。如果想在构造期传递方法，可以使用`setArguments`方法。

## 继承Fragment类（3.0+）

若Fragment需要UI，覆盖onCreateView方法，充气并返回View。

	import android.app.Fragment;
	import android.os.Bundle;
	import android.view.LayoutInflater;
	import android.view.View;
	import android.view.ViewGroup;
	public class MySkeletonFragment extends Fragment {
		@Override
		public View onCreateView(LayoutInflater inflater,
			ViewGroup container, Bundle savedInstanceState) {
			// 如果Fragment没有UI返回null
			// ｛｛第三个参数一定要传且是false｝｝
			return inflater.inflate(R.layout.my_fragment, container, false);
		}
	}

## 使用支持库

本节介绍使用支持库中的Fragment类。兼容Android 1.6以上。

	import android.os.Bundle;
	import android.support.v4.app.Fragment;
	import android.view.LayoutInflater;
	import android.view.ViewGroup;
	
	public class ArticleFragment extends Fragment {
	    @Override
	    public View onCreateView(LayoutInflater inflater, ViewGroup container,
	        Bundle savedInstanceState) {
	        // Inflate the layout for this fragment
	        return inflater.inflate(R.layout.article_view, container, false);
	    }
	}

# 生命周期

Fragment部分生命周期回调与父Activity对应。还有部分自身特有的生命周期回调。例如，当活动的onPause()被调用，则活动内的所有Fragment都将收到onPause()。

![](Fragment_lifetime_callback.png)

注意在子类的回调方法中调用父类的方法。｛｛`onCreateView()`不需要。｝｝

Fragment的生命周期回调方法：

* `void onAttach(Activity activity)`  
活动首次与Activity关联时。
* `void onCreate(Bundle savedInstanceState)`  
初始化fragment。注意此时宿主Activity可能尚未完成onCreate方法。
* `View onCreateView(LayoutInflater inflater, ViewGroup container,
Bundle savedInstanceState)`  
在`onCreate`之后调用。创建Fragment的UI。该方法应该返回已充气的布局。注意，fragment可以没有UI组件。此时返回null。
* `onActivityCreated(Bundle savedInstanceState)`  
在宿主活动完成onCreate后、在Fragment的UI创建后调用。
* `void onStart()`  
当Fragment对用户可见时调用。进入可视生命周期。一般与宿主onStart方法同时调用。
* `void onResume()`  
Fragment对用户可见。进入Active生命周期。一般与宿主onResume方法同时调用。
* `void onPause()`  
可能由于活动被暂停，或Fragment被替换。一般与宿主活动的onPause方法同时被调用
* `void onSaveInstanceState(Bundle savedInstanceState)`  
在Active生命周期结束后，保存UI状态。在父Activity被杀死又重启后，`savedInstanceState`会被传给`onCreate`, `onCreateView`。
* `void onStop()`  
* 活动不再可见。可能由于活动被停止，或Fragment被替换。一般与宿主活动的onStop方法同时被调用。
* `void onDestroyView()`  
onCreateView返回的View与Fragment解除关联后。清理所有与View相关的资源。
* `void onDestroy()`  
当Fragment不再使用时。释放虽有资源，包括线程，数据库等。
* `void onDetach()`  
当Fragment不再与宿主活动关联

由于Fragment可能被动态的添加、删除。因此在父活动活跃的情况下，Fragment自身可能多次经历完整的生命周期。

## Fragment特有的生命周期回调

### 与父Activity关联/解除

Fragment生命周期从绑定到父活动开始，结束于与父活动解除绑定。分别由`onAttach`和`onDetach`表示。若Fragment/Activity已被暂停（退出Active状态），且父Activity的进程被直接终止，则`onDetach`可能不会被调用。`onAttach`事件发生在创建Fragment的UI前，设置发生在Fragment自己和父Activity**完成**初始化前。onAttach一般用于获取对父活动的引用。

### 创建和销毁Fragments

活动的`onDestroy`不保证一定会被调用，同样也不保证Fragment的`onDestroy`被调用。用``方法初始化Fragment。应该在此创建class scoped对象，以确保在Fragment生命周期内它们只被创建一次。

但与活动不同的是，UI不在onCreate方法中创建。

### 创建和销毁UI

Fragment的UI在`onCreateView`和`onDestroyView`中创建和销毁。

利用`onCreateView`初始化Fragment：充气UI，绑定数据到View，创建所需的Service和定时器。返回充气后的View：

	return inflater.inflate(R.layout.my_fragment, container, false);

如果Fragment需要与父Activity的UI交互，等到`onActivityCreated`触发再做。此时活动的UI已构建好。

# 添加到父活动

每个Fragment的实例都必须与父活动关联。这种关联，可以通过在活动的布局文件中定义Fragment实现。但注意，通过布局XML文件向活动添加的Fragment不能在运行时被移除。

> Note: 在API level API 11前，要使用FragmentActivity做父Activity，但在之后可以直接使用常规Activity。  
如果使用v7 appcompat library，活动应该继承ActionBarActivity类（FragmentActivity的子类）。

## 用XML向活动添加Fragment（静态）

下面这个布局文件，当屏幕large时使用，在活动中并排放置两个Fragment。

res/layout-large/news_articles.xml

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="horizontal"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent">
	
	    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
	              android:id="@+id/headlines_fragment"
	              android:layout_weight="1"
	              android:layout_width="0dp"
	              android:layout_height="match_parent" />
	
	    <fragment android:name="com.example.android.fragments.ArticleFragment"
	              android:id="@+id/article_fragment"
	              android:layout_weight="2"
	              android:layout_width="0dp"
	              android:layout_height="match_parent" />
	
	</LinearLayout>

Fragment的实现类可以用`android:name`特性指定，也可以用`class`特性指定。Android内部代码先找`class`特性，找不到才找`android:name`特性。

	<fragment class=”com.example.ExampleFragment”
		android:id=”@+id/example”
		android:layout_width=”match_parent” 
		android:layout_height=”match_parent” />
	</FrameLayout>

在活动中使用该布局：

	import android.os.Bundle;
	import android.support.v4.app.FragmentActivity;
	
	public class MainActivity extends FragmentActivity {
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.news_articles);
	    }
	}

Fragment被充气后，会变成一个View Group。

# Fragment Manager

每个Activity都包含一个Fragment Manager用于管理所包含的Fragments。获取FragmentManager：

	FragmentManager fragmentManager = getFragmentManager();

若使用支持库，用`getSupportFragmentManager()`获得FragmentManager（这是支持库的API）

利用Fragment Manager增加、移除、替换Fragment；访问已添加的Fragment。

可以设置transition动画。

可以指定是否将Transaction放入后退栈。

Fragment Transaction从beginTransaction开始，最后提交。

	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction(); 
	// Add, remove, and/or replace Fragments.
	// Specify animations.
	// Add to back stack if required.
	fragmentTransaction.commit();

> 注意：只有当活动处于resumed状态时，Fragment才可以被添加或移除。

##添加

除了指定Fragment，还需要指定在哪里放置Fragment，即Fragment的父View。还可以指定一个标签，此后，可以利用这个标签和`findFragmentByTag`方法查询到这个Fragment：

	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
	fragmentTransaction.add(R.id.ui_container, new MyListFragment());
	fragmentTransaction.commit();

## 移除

先利用Manager的`findFragmentById`或`findFragmentByTag`方法获取到Fragment。然后将这个实例传入remove()方法：

	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction(); 
	Fragment fragment = fragmentManager.findFragmentById(R.id.details_fragment);
	fragmentTransaction.remove(fragment);
	fragmentTransaction.commit();

## 替换

指定要被替换的Fragment所在的容器ID，指定替换后的Fragment实例。一个可选的标签识别新插入的Fragment。

	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction(); 
	fragmentTransaction.replace(R.id.details_fragment, new DetailFragment(selected_index));
	fragmentTransaction.commit();

## 查找

利用Fragment Manager的`findFragmentById`方法。

	MyFragment myFragment = (MyFragment)fragmentManager.findFragmentById(R.id.MyFragment);

或`findFragmentByTag`：
	
	MyFragment myFragment = (MyFragment)fragmentManager.findFragmentByTag(MY_FRAGMENT_TAG);

没有UI的Fragment只能通过`findFragmentByTag`找到。Because they’re not part of the Activity’s View hierarchy, 它们没有资源标识符，或container resource identifier，因此不能用`findFragmentById`。

# 用Fragment动态布局Activity

对于运行时添加的Fragment，布局中必须有一个Fragment的父View，以容纳Fragment。
例如，下面的布局，一次显示一个Fragment。为了能替换一次显示一个Fragment。活动的布局包含一个空FrameLayout，｛｛注意是Frame不是Fragment！只是一种普通的布局容器！｝｝作为fragment容器。

res/layout/news_articles.xml:

	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/fragment_container"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent" />

如果活动允许Fragment被移除或替换，你应该在活动的onCreate()方法中**初始化**Fragment。**先要检查UI是否已被填充**。若由于配置改变导致Activity被重启，Android会持久化Fragment布局及关联的后退栈。

官方教程的写法：

	import android.os.Bundle;
	import android.support.v4.app.FragmentActivity;
	public class MainActivity extends FragmentActivity {
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.news_articles);
	
			// {{检查容器是否为空的目的是，该活动根据不同配置会有不同布局；}}
			// {{可能某个替换布局采用静态布局，因此没有此容器}}
	        if (findViewById(R.id.fragment_container) != null) {
	            // 如果是从之前的状态恢复，什么也不要做
				// 否则会又创建一个Fragment，遮在上面！
	            if (savedInstanceState != null) {
	                return;
	            }

	            HeadlinesFragment firstFragment = new HeadlinesFragment();
	            
	            firstFragment.setArguments(getIntent().getExtras());
	            
	            // Add the fragment to the 'fragment_container' FrameLayout
	            getSupportFragmentManager().beginTransaction()
	                    .add(R.id.fragment_container, firstFragment).commit();
	        }
	    }
	}

For the same reason, when creating alternative layouts for run time configuration changes, it’s considered good practice to include any view containers involved in any transactions in all the layout variations. Failing to do so may result in the Fragment Manager attempting to restore Fragments to containers that don’t exist in the new layout.

# Fragment、后退栈与生命周期
Fragment替换对UI的改变是巨大的，可以看成是另一个屏幕。因此要允许用户后退。

To allow the user to navigate backward through the fragment transactions, you must call addToBackStack() before you commit the FragmentTransaction.

	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction(); 
	fragmentTransaction.add(R.id.ui_container, new MyListFragment());
	Fragment fragment = fragmentManager.findFragmentById(R.id.details_fragment);
	fragmentTransaction.remove(fragment);
	String tag = null;
	fragmentTransaction.addToBackStack(tag);
	fragmentTransaction.commit();

> Note: 当移除或替换一个Fragment，且事务被放入后退栈中，被移除的Fragment被停止（而不是销毁）。如果用户导航回Fragment，它会被重启。如果不把Transaction放入后退栈，被移除或替换的fragment被销毁。

The `addToBackStack()` method takes an optional string parameter that specifies a unique name for the transaction. The name isn't needed unless you plan to perform advanced fragment operations using the FragmentManager.BackStackEntry APIs.

可以通过程序要求事务退栈，调用`FragmentManager.popBackStack()`，上一个事务会被退栈。

# Fragment Transactions动画

要使用预设的转场动画，调用setTransition方法，传入某个`FragmentTransaction.TRANSIT_FRAGMENT_*` 常量。

	transaction.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);

可以利用`setCustomAnimations`设置定制动画。该方法接受两个animator 资源：one for Fragments that are being added to the layout by this transaction, and another for Fragments being removed:

	fragmentTransaction.setCustomAnimations(R.animator.slide_in_left,
		R.animator.slide_out_right);

The Android animation libraries were significantly improved in Android 3.0 (API level 11) with the inclusion of the Animator class. 于是，传入setCustomAnimations方法的动画资源，不同于使用support library的应用。
Applications built for devices running on API level 11 and above should use Animator resources, whereas those using the support library to support earlier platform releases should use the older View Animation resources.

# 与其他Fragments通信

所有的Fragment-to-Fragment通讯都通过关联的Activity。两个Fragments永远不要相互直接通信。

**定义几个接口**

要允许Fragment与上层活动通信，可以在Fragment中定义一个接口，并在Activity中实现它。在Fragment的onAttach()方法中捕获接口实现，利用此接口与Activity通信。

	public class HeadlinesFragment extends ListFragment {
	    OnHeadlineSelectedListener mCallback;
	
	    // Container Activity must implement this interface
	    public interface OnHeadlineSelectedListener {
	        public void onArticleSelected(int position);
	    }
	
	    @Override
	    public void onAttach(Activity activity) {
	        super.onAttach(activity);
	        
	        // 检查活动是否实现了回调接口
	        try {
	            mCallback = (OnHeadlineSelectedListener) activity;
	        } catch (ClassCastException e) {
	            throw new ClassCastException(activity.toString()
	                    + " must implement OnHeadlineSelectedListener");
	        }
	    }
	    
	    ...
	}

现在 Fragment可以利用onArticleSelected()与活动通讯。下面是Fragment中的代码：

	@Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Send the event to the host activity
        mCallback.onArticleSelected(position);
    }

**实现接口**

活动实现接口：

	public static class MainActivity extends Activity
	        implements HeadlinesFragment.OnHeadlineSelectedListener{
	    ...
	    
	    public void onArticleSelected(int position) {
	        // The user selected the headline of an article from the HeadlinesFragment
	        // Do something here to display that article
	    }
	}

**向Fragment送消息**

宿主活动可以用findFragmentById()捕获Fragment实例，然后直接调用其public方法。

	public static class MainActivity extends Activity
	        implements HeadlinesFragment.OnHeadlineSelectedListener{
	    ...
	
	    public void onArticleSelected(int position) {
	        // The user selected the headline of an article from the HeadlinesFragment
	        // Do something here to display that article
	
	        ArticleFragment articleFrag = (ArticleFragment)
	                getSupportFragmentManager().findFragmentById(R.id.article_fragment);
	
	        if (articleFrag != null) {
	            // If article frag is available, we're in two-pane layout...
	            // Call a method in the ArticleFragment to update its content
	            articleFrag.updateArticleView(position);
	        } else {
	            // Otherwise, we're in the one-pane layout and must swap frags...
	
	            // Create fragment and give it an argument for the selected article
	            ArticleFragment newFragment = new ArticleFragment();
	            Bundle args = new Bundle();
	            args.putInt(ArticleFragment.ARG_POSITION, position);
	            newFragment.setArguments(args);
	        
	            FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
	
	            // Replace whatever is in the fragment_container view with this fragment,
	            // and add the transaction to the back stack so the user can navigate back
	            transaction.replace(R.id.fragment_container, newFragment);
	            transaction.addToBackStack(null);
	
	            // Commit the transaction
	            transaction.commit();
	        }
	    }
	}

# 没有UI的Fragment

没有UI的Fragment可以提供后台功能，且能在Activity重启后保持住。This is particularly well suited to background tasks that regularly touch the UI，或者，配置改变导致Activity重启，维持某些状态。

利用`setRetainInstance`方法，可以令active Fragment保持它的状态，跨Activity重建。调用该发放后，Fragment的生命周期将改变。

Activity被重启后，Fragment不会被销毁、重建，同一个Fragment实例会被保持。父活动被销毁后它会收到`onDetach`事件，新活动被实例化后，接着会调用`onAttach`, `onCreateView`和`onActivityCreated`。｛｛虽然OnDetach方法被调用了，但Fragment实例仍然存在，因此其实例变量仍有效？？？｝｝

> 尽管也可以在一个带UI的Fragment中使用该技术，但这是不推荐的。更好的办法是，将后台任务和所需的状态移到一个新的、不带UI的Fragment。令两个Fragment交互。

下面是无UI的Fragment的骨架：

	public class NewItemFragment extends Fragment {

		@Override
		public void onAttach(Activity activity) {
			super.onAttach(activity);
			// Get a type-safe reference to the parent Activity.
		}
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			// Create background worker threads and tasks.
		}
		@Override
		public void onActivityCreated(Bundle savedInstanceState) {
			super.onActivityCreated(savedInstanceState);
			// Initiate worker threads and tasks.
		} 
	}

To add this Fragment to your Activity, create a new Fragment Transaction, specifying a tag to use to identify it. 因为Fragment没有UI，因此不应该记入到后退栈。

	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
	fragmentTransaction.add(workerFragment, MY_FRAGMENT_TAG);
	fragmentTransaction.commit();

Use the findFragmentByTag from the Fragment Manager to find a reference to it later.

	MyFragment myFragment = (MyFragment)fragmentManager.findFragmentByTag(MY_FRAGMENT_TAG);

# Android Fragment子类

Android SDK包含几个Fragment子类：

* DialogFragment  
A Fragment that you can use to display a floating Dialog over the parent Activity. You can customize the Dialog’s UI and control its visibility directly via the Fragment API. Dialog Fragments参见第10章。
* ListFragment  
A wrapper class for Fragments that feature a ListView bound to a data source as the primary UI metaphor. It provides methods to set the Adapter to use and exposes the event handlers for list item selection. The List Fragment is used as part of the To-Do List example in the next section.
* WebViewFragment  
A wrapper class that encapsulates a WebView within a Fragment. The child WebView will be paused and resumed when the Fragment is paused and resumed. ｛｛WebView也需要被暂停、恢复？？？｝｝

# 例子

* 官方Tutorials：[FragmentBasics.zip](http://developer.android.com/shareables/training/FragmentBasics.zip)


