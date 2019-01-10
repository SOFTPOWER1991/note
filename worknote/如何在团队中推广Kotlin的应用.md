本周工作：

重构首页，完成以下工作：

1. 完成首页整体改版：搜索栏、banner、专区、tab选项卡作为一部分存在于AppBarLayout中；下面的列表都将给ViewPager，后续无论来多少列表都统统交给Viewpager来管理。 
2. 完成列表容器的替换，将listview更换为RecycleView。数据的绑定：Adapter、Holder采用register的方式来绑定，借鉴了MultiType来实现；
3. tab的自定义实现；
4. 关注管理 和 筛选的数据请求以及数据绑定完成；
5. 完成所有holder的迁移工作；
6. 更换下拉刷新和上拉加载的实现方案，列表加载成功和出错时的处理；
7. 更新Project item的UI样式；
8. 首页缓存数据的引入；

下周工作：

1. 首页的统计相关：新版统计、旧版统计；
2. 底部项目曝光个数圆圈按钮的实现；
3. item左滑删除功能；
4. 气泡样式的实现：根据UI给的新版规范来完成；
5. 底部tab的切换相关：红点的处理、统计相关；

重构相关文档：

1. 首页重构细则:   https://c.ethercap.com/pages/viewpage.action?pageId=23291329


