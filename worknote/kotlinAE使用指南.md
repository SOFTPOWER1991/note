Kotlin Android Extensions 是Kotlin团队为android开发者提供的一个Kotlin插件。它可以用来进行视图绑定，简化findViewById()，减少一些复杂的操作。

进行视图绑定时，有如下这几种情况：
* 在Activity中使用；
* 在Fragment中使用；
* 在ViewHolder中使用；
* 在自定义View中使用；

# Kotlin Android Extensions的使用

假如有如下布局文件：

假如有如下布局 `activity_main.xml`：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_height="match_parent"
        android:layout_width="match_parent"
        android:gravity="center"
              >
    <Button
            android:id="@+id/btn_login"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="登录"
    />
</LinearLayout>
```

* 在Module中的build.gradle文件添加插件配置
    
    ```
    apply plugin: 'kotlin-android-extensions'
    ```

*  在需要绑定视图的Activity、Fragment、Adapter及自定义View中引入资源文件
    ```
    import kotlinx.android.synthetic.main.activity_main.*
    ```
* 在使用的位置，直接使用xml中对应的id访问视图，完整代码如下：
    ```
import kotlinx.android.synthetic.main.activity_main.*
    class MainActivity : AppCompatActivity() {
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)
    
            btn_login.setOnClickListener {
                Toast.makeText(this@MainActivity,"登录",Toast.LENGTH_SHORT).show()
            }
        }
    }
    ``` 

引入文件说明：
```
    import kotlinx.android.synthetic.main.activity_main.*
```

* 固定前缀： import kotlinx.android.synthetic.main 
* 布局文件名： activity_main
* * ： 表示引入当前布局下的所有视图view

注意事项：
> 如果在Holder和自定义view中引入，需要在布局文件名后添加View节点，如下：
> ```
> import kotlinx.android.synthetic.main.view_login.view.*
> ```


### 在Activity中使用

在Activity中使用，引入资源文件，直接使用id访问视图

```
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        btn_login.setOnClickListener {
            Toast.makeText(this@MainActivity,"登录",Toast.LENGTH_SHORT).show()
        }
    }
}
```

## 在Fragment中使用

在Fragment中使用，引入资源文件，直接使用id访问视图 有一点特别注意：在onCreateView中不直接访问视图，因为视图没有加载完成，必然引起空指针，需要在onViewCreated中访问视图，代码如下：

```
import kotlinx.android.synthetic.main.view_login.*

class LoginFragment:Fragment() {
    override fun onCreateView(inflater: LayoutInflater?, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        super.onCreateView(inflater, container, savedInstanceState)
        return inflater?.inflate(R.layout.view_login, container, false)
    }

    override fun onViewCreated(view: View?, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        btn_login.setOnClickListener {
            Toast.makeText(context,"登录", Toast.LENGTH_SHORT).show()
        }
    }
}
```

## 在Adapter、ViewHolder中使用

引入布局文件需要添加view节点，可使用ViewHolder中的itemView直接访问视图（当然，也可以在ViewHolder中做一次视图绑定，与传统ViewHolder类似），代码如下：

```
import kotlinx.android.synthetic.main.view_login.view.*

class LoginAdapter(var context: Context):RecyclerView.Adapter<LoginAdapter.ViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(context)
                .inflate(R.layout.view_login,parent,false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.itemView.btn_login.setOnClickListener {
            Toast.makeText(context,"登录", Toast.LENGTH_SHORT).show()
        }
    }

    override fun getItemCount(): Int {
        return 3
    }

    class ViewHolder(view: View) : RecyclerView.ViewHolder(view)
}
```

## 在自定义View中使用

在自定义View中使用，引入布局文件需要添加view节点，在自定义视图中，可直接使用id访问视图，代码如下：

```
import kotlinx.android.synthetic.main.view_login.view.*

class LoginView @JvmOverloads constructor(
        context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : FrameLayout(context, attrs, defStyleAttr) {

    init {
        View.inflate(context,R.layout.view_login,this)

        btn_login.setOnClickListener {
            Toast.makeText(context,"登录", Toast.LENGTH_SHORT).show()
        }
    }
}
```