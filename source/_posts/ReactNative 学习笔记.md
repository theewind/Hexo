---
title: React Native学习笔记
tags: React Native
---

在学习react Native的过程中，对一些知识的记录，不定期更新。。。

1. React 里直接修改 this.state 和调用 setState() 修改 state 的值有什么区别？
---
[https://segmentfault.com/q/1010000002958584](https://segmentfault.com/q/1010000002958584)

使用对this.state赋值并没有什么作用，官方提醒，应该把this.state当成不可变变量。
而使用this.setState方法，会触发异步修改状态，状态改变的同时，会重新执行一次willUpdate，render等流程。需要注意的是，避免在执行完this.setState后马上读取this.state，此操作并不会获得最新修改的状态。

<!--more-->

> Notes:
NEVER mutate this.state directly, as calling setState() afterwards may replace the mutation you made. Treat this.state as if it were immutable.
setState() does not immediately mutate this.state but creates a pending state transition. Accessing this.state after calling this method can potentially return the existing value.
There is no guarantee of synchronous operation of calls to setState and calls may be batched for performance gains.
setState() will always trigger a re-render unless conditional rendering logic is implemented in shouldComponentUpdate(). If mutable objects are being used and the logic cannot be implemented in shouldComponentUpdate(), calling setState() only when the new state differs from the previous state will avoid unnecessary re-renders.

2. react-native-vector-icons
-----
react-native-icons 是 React Native 简单易用的图标组件。

当前支持 4 种字体：

- FontAwesome 4.2 Contains 479 icons
- ionicons 2.0.0 Contains 733 icons, lots of iOS 7 style outlined icons.
- Foundation icons Contains 283 icons.
- Zocial Contains 99 social icons.

因为在RN中没有Button，可以采用react-native-vector-icons替换，包括TabBarItem等。

```
import FontAwesome from 'react-naitive-vector-icons/FontAwesome';
<FontAwesome.TabBarItem
	key = {i}
	title = {controller.title}
	selected = {this.state.selectedTab === controller.title}
	iconName = {controller.icon}
	onPress = {() => {
		this.setState({
			selectedTab: controller.title
		})
	}}
>
<Component navigator = {this.props.navigator} />
</FontAwesome.TabBarItem>

```

```
import Icon from 'react-native-vector-icons/Ionicons';

function TabBarView(props) {
  return (
    <TabBarIOS>
      <Icon.TabBarItem
        title="Home"
        iconName="ios-home-outline"
        selectedIconName="ios-home"
        >
        <View style={styles.tabContent}><Text>Home Tab</Text></View>
      </Icon.TabBarItem>
    </TabBarIOS>
  );
}

```

3. cloneWithRows
---
一般我们都是这样写ListViewDataSource的

```
      let ds = new ListView.DataSource({
        rowHasChanged: (r1, r2) => r1 !== r2
      });
      this.state = {
        dataSource: ds.cloneWithRows(menuCategroys),
      }
      而不是这样：
      this.state = {
          dataSource: new ListView.DataSource({
              rowHasChanged: (row1, row2) => row1 !== row2,
          }),
      }
```
主要是因为，cloneWithRows不仅仅是赋值数据，还会有一些对比操作，判断里面的rowdata是否有更新等操作。
> [How to add/Delete item into ListView?](http://stackoverflow.com/questions/29351259/how-to-add-delete-item-into-listview)

4. onLayout方法
---
**onLayout** *function* 

当组件挂载或者布局变化的时候调用，参数为：

{nativeEvent: { layout: {x, y, width, height}}}

这个事件会在布局计算完成后立即调用一次，不过收到此事件时新的布局可能还没有在屏幕上呈现，尤其是一个布局动画正在进行中的时候。	

```
          onLayout={(event) => {
                    var layout = event.nativeEvent.layout;
                    this.setState({
                        listHeight : layout.height
                    });
                }}

```
对于ListView来说，滑动的过程中不会重新调用onLayout，所以这里的listHeight始终不变，不能用来监控滑动。

5. this.setState
---
this.setState设置一个参数形式如下：

```
<CategoryMenu
                    key={'category_menu'}
                    categories={this.state.newsCategories}
                    fetchNewsList={(category) => {
                        this.setState({
                            currentCategory: category,
                            isRefreshing: true,
                        },  function() {
                            this._fetchNewsList();
                        });
                    }}
                />
```
在CategoryMenu中传入key（默认的props属性），categories获取的分类，已经方法fetchNewsList，而fetchNewList方法主要是更新自身的state，并发送请求，更新state属性，并调用某个方法。因为上述要点1中说过，setState并不会立即生效，调用this.state可能还是就的数据，既可以通过这种方式立即出发自己的某些操作。比如此处的_fetchNewsList，如果currentCategory没有生效，可以把catetory当做参数传递过去。

6. render方法如何根据条件进行渲染
---
在render（）方法里面，经常遇到的就是根据条件进行渲染，特别是对于彼此之间有显示隐藏逻辑，这时候如果把所有view都绘制出来，然后布局，显然比较复杂，这时就可以采用分条件布局的方式。简单说就是先声明一个数组，然后根据各种逻辑，进行push操作，把各种view push进之前的数组中，最后，渲染那个view的数组就可以了。
例如：

```
        let NavigationBar = [];

        // 左边图片按钮
        if (this.props.leftIcon != undefined) {
            NavigationBar.push(
                <TouchableOpacity
                    key={'leftIcon'}
                    activeOpacity={0.75}
                    style={styles.leftIcon}
                    onPress={this.props.leftTouchAction}
                >
                    <Icon color="white" size={30} name={this.props.leftIcon}/>
                </TouchableOpacity>
            )
        }

        // 标题
        if (this.props.title != undefined) {
            NavigationBar.push(
                <Text key={'title'} style={styles.title}>{this.props.title}</Text>
            )
        }

        // 自定义标题View
        if (this.props.titleView != undefined) {
            let Component = this.props.titleView;
            
            NavigationBar.push(
                <Component key={'titleView'} />
            )
        }

        // 右边图片按钮
        if (this.props.rightIcon != undefined) {
            NavigationBar.push(
                <TouchableOpacity
                    key={'rightIcon'}
                    activeOpacity={0.75}
                    style={styles.rightIcon}
                    onPress={this.props.rightTouchAction}
                >
                    <Icon color="white" size={30} name={this.props.rightIcon}/>
                </TouchableOpacity>
            )
        }

        // 右边文字按钮
        if (this.props.rightButton != undefined) {
            NavigationBar.push(
                <TouchableOpacity
                    key={'rightButton'}
                    activeOpacity={0.75}
                    style={styles.rightButton}
                    onPress={this.props.rightButtonAction}
                >
                    <Text style={styles.buttonTitleFont}>{this.props.rightButton}</Text>
                </TouchableOpacity>
            )
        }
        
        return (
            <View style={styles.header}>
                <View style={styles.navigationBarContainer}>
                    {NavigationBar}
                </View>
            </View>
        )
根据条件进行push操作，得到一个渲染列表。
```
还有一种方式是通过，匿名函数的方式，返回一个个的view，如下：

```
const secondLine = (
      <View style = {styles.secondLineView}>
        {(()=>{
          if (spu.recomCount > 0) {
            return (
              <View style style = {styles.rowView}>
                <Image source={require('../../../asset/od_icon_smallzan.png')} style = {styles.imageLike}></Image>
                <Text style = {[styles.detailText,styles.liketext]}>{spu.recomCount}</Text>
              </View>
            );
          }
        })()}
        {(()=>{
          if (spu.dishSales && spu.dishSales.length > 0) {
            return(
              <View>
                <Text style = {styles.detailText}>{spu.dishSales}</Text>
              </View>
            );
          }
        })()}
      </View>
    );
```

7. 字体使用：ionIcons & AwesomeFont
---
ionicons 包含超过 430 个免费漂亮的程序图标。该图标集采用 MIT 许可证，你可在商业应用中使用。它和AwesomeFont的字体就是矢量图,无论是放大还是缩小都不失真的矢量图哦。不仅是在ReactNative中使用，即使是iOS native开发中，也可以使用的。
可以导入FontAwesomeKit框架。更高级的方式，是把字体转换成路径，然后就可以执行各种CoreAnimation的动画了。[参考动画](http://www.tuicool.com/articles/2MJNZn)

而在使用的过程中，只要导入：
	
	import Icon from 'react-native-vector-icons/Ionicons';
	import IconFA from 'react-native-vector-icons/FontAwesome';
	然后自己定义属性：
	{
        key:28,
        title:"iMessage image picker",
        component: Day29,
        isFA: false,
        icon: "navicon-round",
        size:50,
        color:"#48f52e",
        hideNav: false,
      },{
        key:29,
        title:"Push Notifications",
        component: Day30,
        isFA: false,
        icon: "android-notifications",
        size:50,
        color:"#f27405",
        hideNav: false,
      }]
      
      render方法里面调用：
      {elem.isFA? <IconFA size={elem.size} name={elem.icon} style={[styles.boxIcon,{color:elem.color}]}></IconFA>:
              <Icon size={elem.size} name={elem.icon} style={[styles.boxIcon,{color:elem.color}]}></Icon>}
              
8. 如何从listView中获取scrollView的属性
---
需要定义ref

```
  <ListView
          ref="listView"
          dataSource={this.state.dataSource}
          renderRow={this._renderRow
          onScroll={this.handleScroll}
          />
        </View>
```
然后

```
      var scrollProperties = this.refs.listView.scrollProperties;
      console.log(scrollProperties);
      console.log(scrollProperties.offset);
      
      手动滚动到某处
componentDidUpdate() {
    let scrollResponder = this.refs.listView.getScrollResponder();
    scrollResponder.scrollResponderScrollTo({x: 0, y:this.pscroll(), animated: true});
  }
```
通过scrollProperties，然后就可以获取offset了。
还有一种方式是通过：event.nativeEvent.contentOffset.y获得。参考

```
handleScroll: function(event: Object) {

    if (event.nativeEvent.contentOffset.y < -110) { // pull-down
        this.setState({isLoading: true});
        var filter = this.props.filter || this.state.filter;
        // reduce dup fetches
        this.clearTimeout(this.timeoutID);
        this.timeoutID = this.setTimeout(() => this.fetchVideos(filter), 250);
    }
},

```