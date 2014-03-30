# Actitity

## 可以作为Activity的成员变量

* View实例可以。在`onCreate`里通过`findViewById`初始化变量。
* Adapter实例可以。在'onCreate`里初始化变量。

## 工具类/工具方法

在Activity里可以声明一个启动它自己的静态方法：

    public class AddBookActivity extends Activity {
        ...     
        static void show(Context context) {
            final Intent intent = new Intent(context, AddBookActivity.class);
            context.startActivity(intent);
        }
        ...
        
----