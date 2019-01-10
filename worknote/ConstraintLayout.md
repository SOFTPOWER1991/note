1. ConstraintLayout 用来解决什么问题？RelativeLayout 和 LinearLayout的问题

    * 复杂布局能力差，需要不同布局嵌套使用。
    * 布局嵌套层级高。不同布局的嵌套使用，导致布局的嵌套层级偏高。
    * 页面性能低。较高的嵌套层级，需要更多的计算布局时间，降低了页面性能。

    **可以替代其他布局，降低页面布局层级，提升页面渲染性能。**
    
1. 使用：
    添加依赖
    ```
    implementation'com.android.support.constraint ：constraint-layout：1.1.2' 
    ```
    
    
    ==CodeLabs 训练==    
    https://codelabs.developers.google.com/codelabs/constraint-layout-cn/index.html?index=..%2F..%2Fgddchina#3:
    
    属性组理解：
    app:layout_constraintLeft_toRightOf = "@+id/title"
    > 将当前view的左面 放在 title 的右面
    
    自动创建约束条件：
    
    设置居中显示：
    
    ```
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    ```
    
    约束条件偏差：
    
    元素与布局边框之间的间距度量称为——约束条件偏差。
    
    * 元素居中，约束条件偏差为 50%，这表示该元素位于两个边框的正中间。
    * 水平约束条件的偏差更改为 30%，则元素将更接近于左侧边框。
    * 如果它更改为 70%，则元素将更接近于右侧边框。
    * 垂直约束条件也是如此：偏差用于控制元素与上边框和下边框的间距。

    
    更改元素的布局宽度和高度：
    
    * fixed : 指定元素的宽度、高度
    * Match Constraints: 允许元素占用所有可用空间以满足约束条件。（这不同于宽度或高度的match_parent值，此值用于将元素设置为占用父视图的所有可用空间。您不应在 ConstraintLayout 中将match_parent用于任何视图。） （为什么，设置为match_constraints 后 变为了0）
    * wrap_content: 

    约束基线： baseline
    
    打包元素和推断约束条件：
    
    pack: 
    
    infer constraints: 工具用于推断（或确定）与元素的粗略布局相匹配的约束条件。其工作原理是考虑元素的位置和大小。将元素拖动到布局上您需要的位置，然后使用 Infer Constraints 工具自动创建约束条件连接。
    
    # 按比例调整元素大小
    
     app:layout_constraintDimensionRatio="h,1:2"
     
    # 使用分界线对齐大小动态变化的元素
    
    组件宽度大小随着内容区域的变化而变化，导致无法对齐，这时候可以用分界线来处理。
    
    
    Add Vertical barrier和Add Horizontal barrier选项。
    
     <android.support.constraint.Barrier
        android:id="@+id/barrier2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="end"
app:constraint_referenced_ids="settingsLabel,cameraLabel"
        tools:ignore="MissingConstraints"
        tools:layout_editor_absoluteX="411dp" />
        
        
        # 使用链确定多个元素的位置
        
        =======
        
        
        
        
        