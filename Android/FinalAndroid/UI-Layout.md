# 布局

## 定义布局

### 通过代码

	LinearLayout ll = new LinearLayout(this);
	ll.setOrientation(LinearLayout.VERTICAL);
	TextView myTextView = new TextView(this);
	EditText myEditText = new EditText(this);
	myTextView.setText(“Enter Text Below”);
	myEditText.setText(“Text Goes Here!”);
	int lHeight = LinearLayout.LayoutParams.MATCH_PARENT;
	int lWidth = LinearLayout.LayoutParams.WRAP_CONTENT;
	ll.addView(myTextView, new LinearLayout.LayoutParams(lHeight, lWidth));
	ll.addView(myEditText, new LinearLayout.LayoutParams(lHeight, lWidth));
	setContentView(ll);

## FrameLayout

最简单的比较容器是FrameLayout。该容器根本不会排列孩子。他只是将一个孩子堆在另一个上面。定义在后面的在视图上面。FrameLayout常用于创建自定义的可触摸元素。

默认位置在左上角。可以通过`gravity`特性改变位置。

可以使用FrameLayout组合一个按钮和一个ImageView。将按钮背景设为透明。这种方式与设置背景相比，**能对按钮图像的padding和缩放做更好的控制**。

## LinearLayout

5个方面的控制：朝向（Orientation）、大小、weight、gravity和padding。

- 朝向：`android:orientation`决定LinearLayout是行还是列。可以在运行时通过setOrientation()改变朝向。
- 大小：LinearLayout中的所有小部件都要提供`android:layout_width`和`android:layout_height`属性。取值有三种类型：
	- 特定的尺寸，如125dip。
	- `wrap_content`，让控件填满自然的空间。当控件太大时，Android会使用`word-wrap`。
	- `fill_parent`，在考虑完其他控件后，充满容器所有可用空间。注意：API level 8 (Android 2.2)，将`fill_parent`重命名为`match_parent`。
- Weight：例如，我们在一列中有两个多行控件，我们想，在列中其他控件被分配空间后，将剩余空间分配给这两个控件。  
为了达到上述目的，需要`android:layout_weight`。如果为一对小控件设置相同的非零的权重，多出的空间将在它们之间平分。如果一个控件设置1另一个设置2，则第二个占据2/3空间，第一个占据1/3。weight的默认值是0。  
如果让一个容器内的所有控件按百分比分配空间，则：
	- 设置所有小部件的`android:layout_width`为0。
	- 按比例设置`android:layout_weight`，且让所有的weight加起来为100。
- Gravity：默认LinearLayout中元素是左对齐、上对齐的。若想改变对齐方式，设置`android:layout_gravity`（或调用`setGravity()`）。  
对于一列控件，gravity值可以是`left`, `center_horizontal`, `right`。对于一行控件，默认按文本基线对象（baseline）。也可以设为`center_vertical`。
- Margins：只有非透明背景的小部件，padding和margins才有区别。可以设置对立边`android:layout_marginTop`或所有边`android:layout_margin`。

例子

	<?xml version="1.0" encoding="utf-8"?> 
	<LinearLayout 
		xmlns:android="http://schemas.android.com/apk/res/android" 
		android:orientation="vertical" 
		android:layout_width="fill_parent" 
		android:layout_height="fill_parent" > 
		<RadioGroup android:id="@+id/orientation" 
			android:orientation="horizontal" 
			android:layout_width="wrap_content" 
			android:layout_height="wrap_content" 
			android:padding="5dip"> 
			<RadioButton android:id="@+id/horizontal" android:text="horizontal" /> 
			<RadioButton android:id="@+id/vertical" android:text="vertical" /> 
		</RadioGroup> 
			<RadioGroup android:id="@+id/gravity" 
				android:orientation="vertical" 
				android:layout_width="fill_parent" 
				android:layout_height="wrap_content" 
				android:padding="5dip"> 
			<RadioButton 
				android:id="@+id/left" 
				android:text="left" /> 
			<RadioButton 
				android:id="@+id/center" 
				android:text="center" /> 
			<RadioButton 
				android:id="@+id/right" 
				android:text="right" /> 
		</RadioGroup> 
	</LinearLayout>

例子2

	public class LinearLayoutDemo extends Activity 
		implements RadioGroup.OnCheckedChangeListener { 
		RadioGroup orientation; 
		RadioGroup gravity; 
		@Override 
		public void onCreate(Bundle icicle) { 
			super.onCreate(icicle); 
			setContentView(R.layout.main); 
			orientation=(RadioGroup) findViewById(R.id.orientation); 
			orientation.setOnCheckedChangeListener(this); 
			gravity=(RadioGroup) findViewById(R.id.gravity); 
			gravity.setOnCheckedChangeListener(this); 
		} 
		public void onCheckedChanged(RadioGroup group, int checkedId) { 
			switch (checkedId) { 
			case R.id.horizontal: 
				orientation.setOrientation(LinearLayout.HORIZONTAL); 
				break; 
			case R.id.vertical: 
				orientation.setOrientation(LinearLayout.VERTICAL); 
				break;
			case R.id.left: 
				gravity.setGravity(Gravity.LEFT); 
				break; 
			case R.id.center: 
				gravity.setGravity(Gravity.CENTER_HORIZONTAL); 
				break; 
			case R.id.right: 
				gravity.setGravity(Gravity.RIGHT); 
				break; 
			} 
		} 
	}



