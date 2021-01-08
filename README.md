[![](https://jitpack.io/v/wp529/ExposureRecyclerView.svg)](https://jitpack.io/#wp529/ExposureRecyclerView)
### ExposureRecyclerView

​	ExposureRecyclerView是一个很方便对RecyclerView的item进行曝光埋点的库，在现有代码上少量侵入即可实现RV的曝光埋点。支持RV的线性布局、网格布局、瀑布流布局，横向滑动曝光埋点。支持配置item的有效曝光面积。

**添加依赖:**

```groovy
allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
dependencies {
	implementation 'com.github.wp529:ExposureRecyclerView:1.1.0'
}
```

###### 使用方式：

1. 将ItemView的根布局替换为对应的LayoutView，内部提供了三个LayoutView，若需更多的LayoutView可以自行实现IProvideExposureData接口

   ExposureLinearLayout，ExposureFrameLayout，ExposureRelativeLayout

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <com.wp.exposure.view.ExposureFrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:tools="http://schemas.android.com/tools"
       android:id="@+id/exposureRoot"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:orientation="vertical">
       <TextView
           android:id="@+id/tvText"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           tools:text="TEXT" />
   </com.wp.exposure.view.ExposureLinearLayout>
   ```

2. 在adapter为item设置数据时，为LayoutView绑定上曝光的数据

   ```kotlin
   //注:这里的data对象最好实现了equals方法
   exposureRoot.exposureBindData = data
   ```

3. 在给RV设置完adapter后实例化RecyclerViewExposureHelper，实例化时需传递三个参数，需要做曝光埋点的RecyclerView，item曝光状态改变监听器(泛型类型为第二步中data的类型)，item的有效曝光面积判定，eg:设置为50 若item的面积为200平方,则必须要展示200 * 50% = 100平方及以上才算为曝光

   ```kotlin
   recyclerViewExposureHelper =
               RecyclerViewExposureHelper(rvList, 50, object : IExposureStateChangeListener<String> {
                   override fun onExposureStateChange(
                       bindExposureData: String,
                       position: Int,
                       inExposure: Boolean
                   ) {
                       Log.i(
                           "ListActivity", "${bindExposureData}${
                           if (inExposure) {
                               "开始曝光"
                           } else {
                               "结束曝光"
                           }
                           }"
                       )
                   }
               })
   ```

4. 一般情况下在Activity/fragment的onResume和onPause调用即可

   ```kotlin
   //onResume
   recyclerViewExposureHelper.onVisible()
   //onPause
   recyclerViewExposureHelper.onInvisible()
   ```

5. 若RV被嵌套在可滚动控件(eg:ScrollView,NestedScrollView,RecyclerView等)中，将会导致RecyclerViewExposureHelper持有的RV不能响应滑动的情况,就必须由外部告知RV被滚动了触发曝光收集

至此，item曝光埋点接入完成，只需在**onExposureStateChange**回调中进行处理即可。**onExposureStateChange**回调会回传三个参数。item绑定的曝光数据，曝光状态改变的位置，曝光状态。曝光状态true代表item从未曝光状态进入曝光中状态，false代表item从曝光中状态进入结束曝光状态

RecyclerView的item曝光埋点对客户端来说只用处理三个问题,此库的作用即是处理这三个问题

* 可见面积是否为有效曝光

* item可见(开始曝光)

* item不可见(结束曝光)

埋点SDK一般会提供三个api供客户端调用

* onItemExposureStart()

* onItemExposureEnd()

* onItemExposureUpload()

所以只需在特定**onExposureStateChange**调用埋点SDK的api即可,至于曝光时长是否为有效曝光由埋点SDK进行计算

若没有中台(埋点SDK)，也可通过**onExposureStateChange**自行处理相关逻辑

##### 可查看demo获知更多使用细节

