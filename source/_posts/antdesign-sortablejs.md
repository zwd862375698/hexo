---
title: antdesign-vue结合sortablejs实现两个table相互拖拽排序
---
# 实现效果
本来想在网上看看有没有基于antdesign做的，然后发现是真的少啊！废话不多说，先上图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107161521792.gif#pic_center)




## sortablejs介绍
首先先来认识一下这个插件: [sortablejs](http://www.sortablejs.com/index.html)
大家可以去细读一下它的api文档：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107162025864.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3ZDg2MjM3NTY5OA==,size_16,color_FFFFFF,t_70)
这边我就着重介绍一下我用到的api。
1.`group`可以传入对象，参数值为`name`,`pull`,`put`,
 **name**:如果是要两个列表下进行拖动的话，name的值必须为一样；
  **pull**:pull用来定义从这个列表容器移动出去的设置，true/false/'clone'/function
   * true :列表容器内的列表单元可以被移出；
    * false：列表容器内的列表单元不可以被移出；
   *  clone：列表单元移出，移动的为该元素的副本；
  *   function：用来进行pull的函数判断，可以进行复杂逻辑，在函数中return false/true来判断是否移出；
    
**put**:put用来定义往这个列表容器放置列表单元的的设置，true/false/['foo','bar']/function；
* true:列表容器可以从其他列表容器内放入列表单元；
* false：与true相反；
* ['foo','bar']：这个可以是一个字符串或者是字符串的数组，代表的是group配置项里定义的name值；
 * function：用来进行put的函数判断，可以进行复杂逻辑，在函数中return false/true来判断是否放入；
 
2.`animation`  ms, number 单位：ms，定义排序动画的时间；
3. `handle`: 格式为简单css选择器的字符串，使列表单元中符合选择器的元素成为拖动的手柄，只有按住拖动手柄才能使列表单元进行拖动（**你想让哪个元素拖动就绑定这个元素的class**）；
4. `onStart:function(evt){}`开始拖拽的回调方法；
5. `onUpdate:function(evt){}`列表内元素顺序更新的回调方法；
6. `onAdd:function(evt){}`元素从一个列表拖拽到另一个列表的回调方法；
7. `onRemove:function(evt){}` 元素从列表中移除进入另一个列表的回调方法；
这个需求用到这些**api**也就足够了。
## 具体实现
1.第一步先初始化`sortable`方法，因为我们的需求是两个表格拖拽，所以初始化2个方法。
**html**代码
```
<s-table
    ref="table"
    size="default"
    class="left-table"
    rowKey="key"
    :columns="columns"
    :data="loadData">
</s-table>
          
<s-table
    class="sort-table"
    ref="table2"
    size="default"
    class="left-table"
    rowKey="key"
    :columns="columns"
    :data="loadData">
</s-table>
```
具体的*columns* 和*loadData*就不多余阐述。

**JS**代码
```javascript
import Sortable from 'sortablejs'
methods:{
    // 初始化 sortable 实现拖动
    initSortable () {
      var that = this
      var el = this.$el.querySelector('.sort-table tbody')
      Sortable.create(el, {
        handle: '.ant-table-row',
        animation: 150,
        group: { name: 'name', pull: true, put: true },
        onUpdate: function (evt) {
  
        },
        // 开始拖拽的时候
        onStart: function (evt) {
         
        },
        onAdd: function (evt) {
          
        },
        onRemove: function (evt) {
 
        }
      })
    },
     initSortable1 () {
      var that = this
      var el = this.$el.querySelector('.left-table tbody')
      Sortable.create(el, {
        handle: '.ant-table-row',
        animation: 150,
        group: { name: 'name', pull: true, put: true },
        onUpdate: function (evt) {
  
        },
        // 开始拖拽的时候
        onStart: function (evt) {
         
        },
        onAdd: function (evt) {
          
        },
        onRemove: function (evt) {
 
        }
      })
    },
   }
```
关于`handle`所取的**class**，因为我们是要对*antdesign*表格的每一行进行拖拽，所以要选取到他每一行的**class**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107164241136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3ZDg2MjM3NTY5OA==,size_16,color_FFFFFF,t_70)至此两个**table**之间就可以实现拖拽效果，但仅仅只是**拖拽效果**。
因为这样拖拽之后，两边的数据源并没有发生变化，而且明明已经拖拽过来之后，另一边的表格的展示页会存在错误：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107165016844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3ZDg2MjM3NTY5OA==,size_16,color_FFFFFF,t_70)
排序是我右边表格特有的，但是这边的表格是不需要这个排序的，而且如果拖拽成功的话为什么还会显示**暂无数据**呢，最后左边表头的`CheckBox`也无法选中。所以到此为止只是有拖拽效果而已。
2.在拖拽动作之后，把左右两边的数据源重新赋值，这里有两种实现思路：
  * 每一次拖拽之后都去请求后台数据，拿到新的数据源之后重新赋值给表格，
  * 前端自己做好数据源的处理，等所有的拖拽结束之后排好序再给后台保存。
  
 考虑到性能消耗，我就选择了第二种：
 1）定义左右两边的数据源数组
 ```javascript
data(){
	return{
	  unMatchedList: [], // 左边未匹配的数据
      dataList: [], // 右边已匹配的数据
      pullIndex ：''，//原数组拖拽元素的下标
	}
}
```
2)在每一次`remove`或者`add`的时候更新数据源，这里只写了一个表格拖拽的方法，另一个只要把`that.dataList`和`that.unMatchedList`左右两边的数据源赋值调换一下就行，就不贴重复代码了
```javascript
		 // 开始拖拽的时候
        onStart: function (evt) {
          that.pullIndex = evt.oldIndex
        },
        onAdd: function (evt) {
        //evt.newIndex 移入到新数组的下标
        //pullIndex 原数组拖拽元素的下标
           that.dataList.splice(evt.newIndex, 0, that.unMatchedList[that.pullIndex])
           that.dataList.forEach((item, index) => {
            item.sort = index + 1
          })
          //通知table视图更新
          that.$nextTick(() => {
             that.$refs.table2 && this.$refs.table2.refresh(true)
      		 that.$refs.table && this.$refs.table.refresh(true)
          })
        },
        onRemove: function (evt) {
          that.dataList.splice(evt.oldIndex, 1)
          that.dataList.forEach((item, index) => {
            item.sort = index + 1
          })
          that.$nextTick(() => {
            that.$refs.table2 && this.$refs.table2.refresh(true)
      		that.$refs.table && this.$refs.table.refresh(true)
          })
        }
      })
```
3）实现同一个表格上下拖拽排序
```javascript
  initSortable () {
      var that = this
      var el = this.$el.querySelector('.sort-table tbody')
      Sortable.create(el, {
        handle: '.ant-table-row',
        animation: 150,
        group: { name: 'name', pull: true, put: true },
        //这里千万不要用onEnd 方法
        onUpdate: function (evt) {
          var o = evt.oldIndex
          var n = evt.newIndex
          if (o === n) {
            return
          }
          that.sortListAndUpdate(that.dataList, o, n)
        },
      })
    },
    // 对数据进行排序，要求 o（oldIndex） 和 n（newIndex） 从 0开始
    sortList (list, o, n) {
      var newTableData = JSON.parse(JSON.stringify(list))
      var data = newTableData.splice(o, 1, null)
      newTableData.splice(o < n ? n + 1 : n, 0, data[0])
      newTableData.splice(o > n ? o + 1 : o, 1)
      return newTableData
    },
    /**
     * 对数据排序并更新 table， 要求 o（oldIndex） 和 n（newIndex） 从 0开始
     */
    sortListAndUpdate (list, o, n) {
      var newTableData = this.sortList(list, o, n)
      newTableData.forEach((item, index) => {
        item.sort = index + 1
      })
      this.$nextTick(() => {
        this.dataList = newTableData
        that.$refs.table2 && this.$refs.table2.refresh(true)
      })
    },
```
这边我们选用`onUpdate`方法来排序，不要用`onEnd`方法，因为只要你有拖拽效果，都会去触发`onEnd`方法，导致左右拖拽完后又会触发一次排序。
欢迎大家给我提问啦~~