---
categories:
  - Hexo
tags:
  - hexo
  - stellar主题效果
cover: theme
poster:
  headline: Stellar主题标签效果测试
  color: red
abbrlink: f320e422
---

Stellar 主题标签的效果测试。此效果与写法格式为 Stellar 1.18.0 版本。

<!-- more -->

{% quot 我很喜欢你 像风走过八百里地 不问归期 %}

{% quot 青梅煮酒论天下 el:hashtag %}

{% quot 一、Note标签测试（带标题与不带标题） el:h1 %}

{% note 我这一辈子，苦没有真正苦过，爱没有用力爱过。每天受着信息大潮的冲击，三观未定又备受曲折。贫穷不再是正义，又妄图不让金钱成为唯一的追求。过早看到了更大的世界，勤奋却又不过三天。热血透不过键盘和屏幕，回忆止于游戏和高考。像一群没有根的孩子，在别人的经历和精神里吵闹 color:green %}

{% note color:green 换行测试(实际采用换行写法，可以看到下面内容并没有换行) 
我这一辈子，苦没有真正苦过，爱没有用力爱过。
每天受着信息大潮的冲击，三观未定又备受曲折。
贫穷不再是正义，又妄图不让金钱成为唯一的追求。
过早看到了更大的世界，勤奋却又不过三天。
热血透不过键盘和屏幕，回忆止于游戏和高考。
像一群没有根的孩子，在别人的经历和精神里吵闹 %}

{% note color:orange 苍云 有人居庙堂，有人守边疆。 %}

> 个人推荐使用 ablock 标签代替 note，如果很简单的情况：例如 md 文件的引用 > 可以使用 note.

{% quot 二、ablock标签测试 el:h1 %}

{% ablock ablock 标签测试 color:green %}
我这一辈子，苦没有真正苦过，爱没有用力爱过。

每天受着信息大潮的冲击，三观未定又备受曲折。

贫穷不再是正义，又妄图不让金钱成为唯一的追求。

过早看到了更大的世界，勤奋却又不过三天。

热血透不过键盘和屏幕，回忆止于游戏和高考。

像一群没有根的孩子，在别人的经历和精神里吵闹。
{% endablock %}

{% quot 三、ablock + tabs 嵌套标签测试 el:h1 %}

{% ablock %}
{% tabs %}
<!-- tab 代码 -->
{% codeblock rust lang:rs %}
struct User {
    name: String,
    age: i32,
}
impl User {
    pub fn new(name: String, age: i32) -> User {
        ...
    }
}
{% endcodeblock %}

<!-- tab 彩色代码块 -->

{% ablock child:codeblock color:green %}
{% codeblock lang:js %}
async function getResponse() {
    ...
}
{% endcodeblock %}
{% endablock %}

{% ablock child:codeblock color:green %}
{% codeblock lang:java %}
public static final ConcurrentHashMap map = null;
{% endcodeblock %}
{% endablock %}

{% endtabs %}
{% endablock %}

{% quot 四、单独 codeblock 标签测试 el:h1 %}

{% codeblock lang:java %}
public static final ConcurrentHashMap map = null;
{% endcodeblock %}

{% quot 五、grid 分侧普通标签 el:h1 el:h1 %}

{% grid bg:block %}

<!-- cell left -->
<center>任凭世人笑我癫</center>
<center>狂叹红尘游世间</center>

<!-- cell right -->

<center>空悬北斗点将勺</center>
<center>运筹群星指天枢</center>

{% endgrid %}


{% quot 六、grid 分侧卡片标签 el:h1 %}

{% grid bg:card %}

<!-- cell left -->

<center>没有决断之心的舍弃过往，不过是故作姿态罢了</center>

<!-- cell right -->

<center>神蛊温皇要毁，不但要毁掉过去，还有现在、未来。</center>

{% endgrid %}

{% quot 七、folding 折叠块标签测试 el:h1 %}

{% folding 隐藏起来的块你看到了嘛 child:codeblock open:false color:red %}
fn main {
    println!("{}", "Hello World")
}
{% endfolding %}

{% quot 五、timeline 时间线标签 el:h1 %}

{% timeline %}
<!-- node 2022 年 10 月 28 日 -->
在加班
<!-- node 2022 年 10 月 29 日 -->
在加班
<!-- node 2022 年 10 月 30 日 -->
在加班
{% endtimeline %}

{% quot 六、swiper 轮播标签 el:h1 %}

{% swiper width:max %}
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot11.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot12.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot13.png)
{% endswiper %}

{% swiper width:min %}
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot11.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot12.png)
![](https://fastly.jsdelivr.net/gh/cdn-x/wiki@1.0.2/prohud/screenshot13.png)
{% endswiper %}

{% quot 七、单选、复选样式标签 el:h1 %}
{% radio color:red 没有勾选的单选框 %}
{% radio color:yellow checked:true 已勾选的单选框 %}
{% checkbox 没有勾选的复选框 %}
{% checkbox checked:true已勾选的复选框 %}
{% checkbox symbol:plus color:green checked:true 加号的绿色的已勾选的复选框 %}
{% checkbox symbol:minus color:yellow checked:true 显示为减号的黄色的已勾选的复选框 %}
{% checkbox symbol:times color:red checked:true 显示为乘号的红色的已勾选的复选框 %}

{% quot 八、folders 折叠组 el:h1 %}

{% folders %}
<!-- folder 你最喜欢的食物 -->
辣椒
<!-- folder 你最喜欢的人 -->
女神
<!-- folder 你最喜欢的事 -->
coding
{% endfolders %}
