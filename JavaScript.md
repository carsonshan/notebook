

## 使用localStorage

Cookie存数据影响访问速度(每次请求都需要带上Cookie)，使用localStorage存储有更大容量，还不易丢失

建议将用户的大段输入随时存储到localStorage中

高级应用可以是把js等代码文件这样缓存到本地，安全性讨论见[https://imququ.com/post/enhance-security-for-ls-code.html](https://imququ.com/post/enhance-security-for-ls-code.html)

```
//写入
var storage=window.localStorage;
storage["a"]=1;
//清空
window.localStorage.clear();
```

----

## 使用phantomjs爬取网页

有些时候我们用Python的requests并不能很完美地渲染好网页，例如人家用酷炫的js作图了，我就想得到这张图，这时候用phantomjs就好啦

爬取目标：

[http://oncokb.org/#/gene/AKT1](http://oncokb.org/#/gene/AKT1)

这个网页的右边有一张Tumor Types with AKT1 Mutations的图

代码：

[code/spider.oncokb.js](code/spider.oncokb.js)

代码的细节：

1. 打开页面之前为了截图方便需要先设置浏览器的大小，这里设置为了1920*1080

2. 不要一打开页面就截图，而是等到页面加载好了最后一个请求(从Chrome开发人员工具查看最后的请求是啥)后，再等待5s后执行截图、导出HTML并退出

3. 为了防止无限等待，设置最长2min后timeout退出

4. 为了方便批量化处理，从命令行参数读取需要爬取的基因名称

5. 在运行的时候有设置代理和不要载入图片的参数，具体见[官方文档](http://phantomjs.org/api/command-line.html)

----

## jQuery劫持show事件

我的需求：用户登录的div需要点击Login后显示(toggle)，此时浏览器已经自动帮用户填上了用户名和密码，用户需要手动点击登录按钮才会触发登录请求；现在我想加入快速登录功能，在显示登录div后自动提交登录请求，如果为空或密码错误再交给用户输入

我的解决方案：加入下述扩展jQuery的代码后，对#login绑定beforeShow事件，处理函数先根据全局变量是否存在来判断是否执行过（防止死循环），如果没有执行过则执行登录函数clicklogin并设置全局变量

效果：如果浏览器自动填入了正确的用户名密码，则用户点击Login后快速闪过登录输入框即完成登录；如果浏览器没有自动填入用户名密码，clicklogin函数直接return，用户没有感知；如果浏览器填入的密码是错的，用户会看到密码错误提示，1s后再次toggle登录的div要求用户输入

From: http://stackoverflow.com/questions/1225102/jquery-event-to-trigger-action-when-a-div-is-made-visible

引入jQuery后，修改jQuery自身的show函数以扩展bind：

```
jQuery(function($) {

  var _oldShow = $.fn.show;

  $.fn.show = function(speed, oldCallback) {
    return $(this).each(function() {
      var obj  = $(this),
          newCallback = function() {
            if ($.isFunction(oldCallback)) {
              oldCallback.apply(obj);
            }
            obj.trigger('afterShow');
          };

      // you can trigger a before show if you want
      obj.trigger('beforeShow');

      // now use the old function to show the element passing the new callback
      _oldShow.apply(obj, [speed, newCallback]);
    });
  }
});
```

然后就可以使用bind注册`beforeShow`，`afterShow`咯：

```
jQuery(function($) {
  $('#test')
    .bind('beforeShow', function() {
      alert('beforeShow');
    }) 
    .bind('afterShow', function() {
      alert('afterShow');
    })
    .show(1000, function() {
      alert('in show callback');
    })
    .show();
});
```

----

## 读取GET参数

有些时候对GET参数的处理交给了前端，后端的PHP可以$_GET["parameter"]，前端JS咋办呢？

From: http://stackoverflow.com/questions/979975/how-to-get-the-value-from-the-get-parameters

```
var QueryString = function () {
  // This function is anonymous, is executed immediately and 
  // the return value is assigned to QueryString!
  var query_string = {};
  var query = window.location.search.substring(1);
  var vars = query.split("&");
  for (var i=0;i<vars.length;i++) {
    var pair = vars[i].split("=");
        // If first entry with this name
    if (typeof query_string[pair[0]] === "undefined") {
      query_string[pair[0]] = decodeURIComponent(pair[1]);
        // If second entry with this name
    } else if (typeof query_string[pair[0]] === "string") {
      var arr = [ query_string[pair[0]],decodeURIComponent(pair[1]) ];
      query_string[pair[0]] = arr;
        // If third or later entry with this name
    } else {
      query_string[pair[0]].push(decodeURIComponent(pair[1]));
    }
  } 
  return query_string;
}();
```

执行后就可以这么使用：

```
if (typeof(QueryString.parameter)!="undefined") {
    alert(QueryString.parameter);//do something with the parameter
}
```

----

## 使用 Github Issue 作为博客评论区

人家大佬的项目：[http://github.com/wzpan/comment.js](http://github.com/wzpan/comment.js)，[中文文档](http://www.hahack.com/codes/comment-js/)

如果觉得cloudflare加载速度不佳，可以把所有js打包成一个文件

效果如本博客页面底部评论区所示，为了偷懒就没有为每个md文件单独开issue了，整个blog共用一个issue

----

## history.replaceState修改历史记录

如v2ex按照是否:visited来区分点开过和没点开过的帖子，其实现是url带上#reply回复数量

但如果帖子页面有多种进入方式（自动跳转到页尾、发起了回复等），那么url并不一定与需要的一致

我们可以使用history API来修改历史记录，从而保证带上`#reply回复数量`的url一定被认为访问过；而且自动改回去用户无感知（否则刷新后会打开不一样的页面）

代码如下：

```
<script>
setTimeout( function(){
    var oldurl = location.href;
    history.replaceState(null, null, '/t/{{topic["id"]}}#reply{{topic["replyCount"]}}');
    history.replaceState(null, null, oldurl);
}, 1000);
</script>
```

----

## 记住一个checkbox的状态（用localStorage）

查询是否勾选用`.is(":checked")` ， 改变勾选状态用`.prop("checked",true)`

```
<script>
function checkbox_onclick(){
    var checked = $("#thecheckbox").is(":checked");
    if(checked) localStorage.setItem("status_thecheckbox","1");
    else localStorage.setItem("status_thecheckbox","0");
}
</script>
<input type="checkbox" id="thecheckbox" onclick='checkbox_onclick();'>
<script>
    var status_thecheckbox = localStorage.getItem("status_thecheckbox");
    if(status_thecheckbox!=null && status_thecheckbox=="1") $("#thecheckbox").prop("checked",true);
</script>
```

----

## NodeJS

### 用Docker执行npm

例如安装canvas和gifencoder包：

```
PACKAGES="canvas gifencoder"
docker run --rm --volume="`pwd`:/app" -w /app -it node:10 npm install ${PACKAGES}  --registry=https://registry.npm.taobao.org
```

----

## 使用InstantClick踩坑

### 快速使用

http://instantclick.io/v3.1.0/instantclick.min.js

一定要在页面底部 `</body>`之前才能引入：

```
<script src="instantclick.min.js" data-no-instant></script>
<script data-no-instant>InstantClick.init('mousedown');</script>
```

### 被预加载的页面不能让后端返回302

否则会显示跳转之前的URL

这种情况下可以对这个链接禁止预加载（不过更应该考虑这种链接改为post请求） 在a标签加上`data-no-instant`

### 注意默认配置下后端将被频繁请求 频率限制需要放宽

[官网](http://instantclick.io/download)给出的代码使用`InstantClick.init()`，意味着鼠标移动上去就会触发加载（不是只触发一次），鼠标反复移动会导致大量的请求

如果后端做了请求频率限制 需要放宽限制

还是改为用`mousedown`来初始化 只有用户确实点击了才开始加载 据说也能有很好的效果

### InstantClick引入一些副作用 对页面js要进行修改

#### js无法取得正确的referrer

页面加载的请求是js执行的 document.referrer不会被设置为上一页

#### document.addEventListener 重复触发

例如绑定paste事件 你可能这么写：

```
document.addEventListener('paste', handlepaste);
```

在切换页面后 这个事件会多次绑定 导致多次触发

我的做法是先判断一个变量是否存在 不存在才设置：

```
if(typeof paste_registered == "undefined"){
    document.addEventListener('paste', handlepaste);
    paste_registered = true;
}
```

你也可以把这一部分**不能重复执行**的代码放入`<script data-no-instant>`中，但如果前一页没有这一块代码（也就是这个代码是当前页面才有的，需要执行一次），进入当前页面是**不会触发**的

#### 返回上一页重复执行页面添加元素的js 导致元素重复出现

现在的方法是对js动态添加的元素加个class 然后用jQuery的remove方法先通通删掉再添加

#### 页面ready事件不会触发

需要加入`InstantClick.on('change', callback);` 加到Init后即可

但是似乎这个事件触发在页面图片加载完成之前Orz 不够完美

### 超链接的#hash定位功能也需要自己实现

预加载的页面总是定位到顶部，忽视地址栏中的`#end`这种定位hash

我的做法是这样写上述onchange的callback函数`implement_hashjump`：

```
function has_hashjump(){ // if there is a #hash present for jumpping, return true
    var hash = document.location.hash.replace("#","");
    if(!hash) return false;
    if(document.getElementById(hash) || document.getElementsByName(hash).length>0) return true;
    else return false;
}

function implement_hashjump() {
    if ( has_hashjump() ) {
        var hash = document.location.hash.replace("#","");
        if(document.getElementById(hash)) {
            document.documentElement.scrollTop = $("#"+hash).offset().top;
        }
        else{
            document.documentElement.scrollTop = $("[name='"+hash+"']").offset().top;
        }
    }
}
```