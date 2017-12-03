# JS解惑-history

本文不是讲history的replaceState和pushState这2个方法怎么用，而是要说在无法删除浏览器history的情况下，如何能正确回退页面的问题。

## 场景

有个H5的需求，需要一屏一屏的切换页面，到最后一个结果页面时，如果点击了手机的返回键，或者浏览器自带的返回键，希望能回到主页。但是在中间的切屏过程中，点返回或者前进，就是这几个页面之间切换。

比如，总共有这么几个页面：

1. 首页
1. 1/4页
1. 2/4页
1. 3/4页
1. 4/4页
1. 结果页

如果你在2/4页时，点击返回是要回到1/4页，如果你是在结果页，点击返回，是要回到首页。

## 处理

典型的spa应用，通过改变 `location.hash` 并且监听 `onhashchange` 事件，处理对应的页面即可。

## 问题

对于浏览器而言，改变了location.hash，其实也是给浏览器的history对象，写入了一条历史记录。

所以，对于我这个应用的处理，其实就生成了6条历史记录，可以通过：history.length查看（假如浏览器地址栏输入地址直接进入的首页）。

那么，对于需求来说，在中间的页面切换都没有问题，但是到了 `结果页` 时，默认返回是到了 `4/4页`页，如果要回到 `首页` 需要回退好多次。更麻烦的问题是，如果这每一页有参数携带的话，比如我从1/4的参数一直携带到4/4页，最后提交的时候，才取出数据使用，但是如果我直接从3/4页开始下一步，那么前2页的数据就携带不上了，最终结果可能报错。

## 思路

1. 首先，我想到的办法是删除历史记录，也就是到了结果页，删除前4页的历史记录，这样按返回键的时候，直接就回到首页了。**但是**可惜，history根本没有删除的方法。
1. 调查了半天貌似只有history.go(-n)这个函数回退页面了。

## 经验

关于history的几个知识点：

1. history.length ， 你前进了几页，值就等于几，但是返回的时候这个值不变，除非你再次前进，才会更新。
	* 比如：你前进了5页，这时候length=5，又回退了3页，这时候length=5（理论上应该是2），但是只要你再前进一页，这时候length=3，也就是重新前进时会更新length。
	* 所以，使用length的时候一定要注意以上问题。
1. history.go(-n)，相当于回退页面
1. history.back() 和 history.forward() 分别表示返回上一页，进入下一页，如果没有历史记录了，则没有任何操作。
1. 刷新浏览器不会影响历史记录

## 解决

我建立了3个页面，index.html,page.html,result.html，其中page.html使用hash控制4个页面。

在page.html中做了判断，如果location.hash 不为空，则表示刷新了页面，或者Back回来的，这时候就执行history.go(-n)

伪代码：

```js
//page.js
//页面加载时判断
//以下逻辑可以保证，假如当前在page3，刷新后就回退3页，就到首页了；
//如果是从result.html返回来的，那么是page4，刷新后，也回到首页了；
//但是，如果在page.html页面内部，只需要控制好onhashchange对应的模块显示/隐藏即可；
switch(hash){
	case 1 : history.go(-1);break;
	case 2 : history.go(-2);break;
	case 3 : history.go(-3);break;
	case 4 : history.go(-4);break;
}

//result.js
//页面上有返回首页的按钮时，千万不能用：location.href='index.html'，不然用户点击浏览器自带的返回按钮时，又会错乱。
//应该用history.go来处理，只有这种处理了，回到首页后，如果用户用浏览器自带的返回键，是会回到进入首页的页面的，不然又会回到结果页了。
$('#backHome').click(function(){
	history.go(-5);
});

```

> 以下总结感觉纯属扯淡，讲的比较啰嗦，估计一般人没耐心看完，不过如果你恰好遇到这类型问题了，估计还是会有所启发吧。