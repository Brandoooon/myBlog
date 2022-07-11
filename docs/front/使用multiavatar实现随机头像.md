Multiavatar是一个开源免费的随机头像生成器。

官网：

https://multiavatar.com/

Github：

https://github.com/multiavatar/Multiavatar

![image-20220709170801099](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/front/img/image-20220709170801099.png)

Multiavatar通过字符串随机生成一个头像，可以选择.png格式或者.svg格式，使用方法主要有两种。

一是官方提供的API，使用下面的链接，红框内可以改为任意字符串，api会返回一个png或svg格式的图片，可以直接在html中使用，例如img元素的src属性。

![image-20220709171250139](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/front/img/image-20220709171250139.png)

缺点是如果字符串是中文，经过html编码可能导致无法正常获取头像。

所以推荐第二种方式：

![image-20220709171844318](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/front/img/image-20220709171844318.png)

这里以原生js为例来演示一下：

下载multiavatar.min.js并引入到项目中。

````javascript
<script src="multiavatar.min.js"></script>

<script>
  var avatarId = 'Binx Bond';
  var svgCode = multiavatar(avatarId);
</script>
````

注意这里svgCode返回的是一个svg图片，而不是图片的链接，所以无法像img的src属性赋值那样来使用。

svg格式如下：

````
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
    <frontrcle cx="100" cy="50" r="40" stroke="black" stroke-width="2" fill="red" /> 
</svg>
````

要在html中嵌入svg，可以通过<embed>、<object>、<iframe>标签，也可以直接在HTML嵌入svg代码，例如：

````
<div>
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1">
        <frontrcle cx="100" cy="50" r="40" stroke="black" stroke-width="2" fill="red" /> 
    </svg>
</div>
````

在我的项目中，我为这个div容器添加了一个name属性，用于接收用户名，然后通过用户名来生成不同的头像，代码如下：

````
<div class="avatar" th:name="${plan.createdBy}"></div>
````

````javascript
function refreshAvatar() {
    var avatars = document.querySelectorAll('.avatar');
    avatars.forEach(function (avatar) {
        let creatorName = avatar.getAttribute('name');
        let svgCode = multiavatar(creatorName);
        $(avatar).html(svgCode)
    })
}
````

效果如下：

![image-20220709172810413](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/front/img/image-20220709172810413.png)