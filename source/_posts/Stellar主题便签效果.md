---
title: 主题标签效果
categories: 主题效果
---


<!-- Stellar 主题标签效果测试 -->

## 1. 居中引用和标题

{% quot Stellar 是最好用的主题 %}

{% quot 热门话题 icon:hashtag %}

{% quot h2标题效果 el:h2 %}

## 2. Note 备注块标签

{% grid 绿色备注块效果 color:green codeblock:true %}
我们这群人，苦没有真正苦过，爱没有用力爱过。

每天受着信息大潮的冲击，三观未定又备受曲折。

贫穷不再是正义，又妄图不让金钱成为唯一的追求。

过早看到了更大的世界，勤奋却又不过三天。

热血透不过键盘和屏幕，回忆止于游戏和高考。

像一群没有根的孩子，在别人的经历和精神里吵闹。
{% endgrid %}

## 3. 折叠代码块

{% folding open:true color:orange 默认打开的折叠代码块 %}
颜色是orange，默认打开的可折叠代码块：

function name() {
    print("hello world");
}
{% endfolding %}

{% folding open:false color:red 默认关闭折叠代码块 %}
颜色是red，默认打开的可折叠代码块：

() =>  {
    print("hello world");
}
{% endfolding %}

{% folding child:codeblock open:false color:red test %}
昨夜星辰昨夜风，话楼西畔桂堂东。
{% endfolding %}

## 4. 复选样式标签

{% checkbox 普通的紫色复选框 checked:true color:purple %}
{% checkbox 加号的红色复选框 checked:true color:red symbol:plus %}
{% checkbox 减号的黄色复选框 checked:true color:yellow symbol:minus %}
{% checkbox 乘号的橙色复选框 checked:true color:orange symbol:times %}

## 5. 时间线标签：

{% timeline %}
<!-- node 2022.01.01 -->
今天是2022年的第一天...
<!-- node 2022.01.02 -->
今天是2022年的第二天...
{% endtimeline %}

## 6. Github 标签

{% ghcard prettywinter/prettywinter.github.io theme:dark %}

## 7. 分栏、代码块标签综合使用

{% tabs active:1 align:center %}
<!-- tab 图片 -->
{% image https://fastly.jsdelivr.net/gh/volantis-x/cdn-wallpaper-minimalist/2020/025.jpg width:300px %}
<!-- tab 表格 -->
|a|b|
|--|--|
|a1|b1|
|a2|b2|
<!-- tab 代码块 -->
{% codeblock lang:js %}
let a = 123
console.log("a的值：", a)
{% endcodeblock %}
{% endtabs %}

## 8. 轮播标签

### 8.1 最大化轮播

{% swiper width:max %}
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot11.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot12.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot13.png)
{% endswiper %}

### 8.2 最小化轮播

{% swiper width:min %}
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot11.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot12.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot13.png)
{% endswiper %}

## 9. 复制标签

{% copy width:max git:ssh jhlzlove/jhlzlove.github.io %}
