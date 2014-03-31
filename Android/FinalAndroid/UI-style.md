## UI样式

### style: 抽象公共样式

若在布局XML中有一些公共的样式，例如，表单标签都具有下面的样子。可以将它们抽象到`style.xml`中。

    <style name="form_label">
        <item name="android:layout_width">wrap_content</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:gravity">right</item>
        <item name="android:paddingRight">10dp</item>
    </style>

于是在布局XML中，可以应用此样式：

    <TextView
        style="@style/form_label"
        android:text="@string/proname" >
    </TextView>

可以覆盖样式中的特性。