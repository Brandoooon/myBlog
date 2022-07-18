# flex布局之固定侧边栏

实现左侧边栏leftBar固定，不随着页面上下滚动而超出页面。

html如下：

````html
<div id="plan-content">
        <div id="leftBar">
        </div>
        <div id="article">
        </div>
    </div>
````

css如下：

父元素不要设置overflow属性；

需固定的子元素设置align-slef: flex-start，并且top:0。

````css
#plan-content{
    height: auto;
    width: 100%;
    top: auto;
    display: flex;
    background: #FAFAFA;
}
#leftBar{
    width: 20%;
    height: auto;
    position: -webkit-sticky;
    position: sticky;
    top: 0;
    align-self: flex-start;
}
#article{
    height: auto;
    width: 80%;
}
````

