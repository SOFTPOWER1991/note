



在我们的绘制渲染机制里面比较耗时的：
1.CPU计算时间
	CPU的优化，从减轻加工View对象成Polygons和Texture来下手
	View Hierarchy中包涵了太多的没有用的view，这些view根本就不会显示在屏幕上面，
	一旦触发测量和布局操作，就会拖累应用的性能表现。

	1.如何找出里面没用的view呢？或者减少不必要的view嵌套。
	工具：Hierarchy Viewer检测

	优化：
		1）当我们的布局是用的FrameLayout的时候，我们可以把它改成merge
			可以避免自己的帧布局和系统的ContentFrameLayout帧布局重叠造成重复计算(measure和layout)
	ViewStub：当加载的时候才会占用。不加载的时候就是隐藏的，仅仅占用位置。

	[hierarchyviewer]Unable to capture data for node
	android.widget.LinearLayout@e6fdb11 in window com.example.android.mobileperf.render/com.example.android.mobileperf.render.ChatumLatinumActivity on device 192.168.56.101:5555

	三个圆点分别代表：测量、布局、绘制三个阶段的性能表现。
	1）绿色：渲染的管道阶段，这个视图的渲染速度快于至少一半的其他的视图。
	2）黄色：渲染速度比较慢的50%。
	3）红色：渲染速度非常慢。

	优化思想:查看自己的布局，层次是否很深以及渲染比较耗时，然后想办法能否减少层级以及优化每一个View的渲染时间。


2.CPU将计算好的Polygons和Texture传递到GPU的时候也需要时间
	OpenGL ES API允许数据上传到GPU后可以对数据进行保存，做了缓存。

3.GPU进行格栅化
	优化：尽量避免过度绘制（overdraw）
	GPU如何优化：
		1.背景经常容易造成过度绘制。
		手机开发者选项里面找到工具：Debug GPU overdraw
		由于我们布局设置了背景，同时用到的MaterialDesign的主题会默认给一个背景。
		/解决的办法：将主题添加的背景去掉

		2.自定义控件如何处理过度绘制。
		可以通过裁剪来处理。



