# [浏览器如何工作](http://taligarsiel.com/Projects/howbrowserswork1.htm#Introduction)

## 1. 从地址栏输入"google.com"到屏幕上看到页面发生了什么？

## 2. 浏览器基本组件

* 用户界面（地址栏，后退\前进，书签）
* 浏览器引擎（查询和操作渲染引擎）
* 渲染引擎（渲染html/css等内容）
* 网络（网络请求）
* UI后端（组合框/窗口等部件）
* JavaScript解析器
* 数据存储（h5新定义了"web数据库"）
  
## 3. 浏览器解析html的算法（简单版）

* 标记化算法，遇到"<"字符，状态更改为"标记打开状态"，接下来接收到的字符会是状态跳转至"标记名称状态"，直到消耗掉">"字符，其中"标记名称状态"的字符会被附加到新标记名称中。之后状态更改回"数据状态"，标签中的内容将会创建和发出字符标记，直到遇到"</"中的"<"，此时重新回到"标记打开状态"，下一个"/"将导致创建"结束标记令牌"并且移动到"标记名称状态"，继续直到">"，然后返回"数据状态"。
* 树构造算法，该算法的输入来自于标记化算法的每一个标记。初始状态为"initial mode"，接收到"html"标记的话，状态变为"before html"，此时会创建一个HTMLHtmlElement节点，并且添加到#document根节点中。然后状态变为"before head"，当接收到一个"body"令牌时候，将会创建一个HTMLHeadElement节点并添加到dom树，此时状态变为"in head"然后变为"after head"，此时会创建一个HTMLBodyElement节点并且添加到dom树，状态变为"in body"，然后接受上述的字符标记（也就是内容），此时会创建一个TextNode节点，字符标记会添加到TextNode节点中。最后，变为"after body"状态，接收"html"结束标签，完成树结构创建。
* 解析完成后，浏览器标记文档为交互式，并且开始解析脚本，然后触发"onload"事件。
* 浏览器容错（webkit容错示例）
  \</br>与\<br>，webkit将统一视为\<br>。
  
  ```js
  if（t-> isCloseTag（brTag）&& m_document-> inCompatMode（））{
      reportError（MalformedBRError）;
      t-> beginTag = true;
  }
  ```

  嵌套混乱的table标签
  
  ```html
  <table>
    <table>
      <tr><td>内表</td></tr>
    </table>
  <tr><td>外表</td></tr>
  </table>
  ```

  webkit将其渲染为两个table
  
  ```html
  <table>
    <tr><td>内表</td></tr>
  </table>
  <table>
    <tr><td>外表</td></tr>
  </table>
  ```

  标签嵌套层次太深，webkit将只允许最多20层的相同元素嵌套
  
## 4. 浏览器解析css的算法（简单版）

css词法正则定义

词法名称|词法含义|表达式
---|:--:|--:
comment|注释|\/\*[^*]*\*+([^/*][^*]*\*+)*\/
num|数字|[0-9]+|[0-9]*"."[0-9]+
nonascii|非ascii码|[\200-\377]
nmstart|xx|[_a-z]|{nonascii}|{escape}
nmchar|xx|[_a-z0-9-]\|{nonascii}|{escape}
name|名称|{nmchar}+
ident|标识符|{nmstart}{nmchar}*

&emsp;&emsp;webkit使用flex和bison解析器生成器解析css，每个css文件都被解析为StyleSheet对象，每个对象都包含css规则，每条CSS规则对象包含选择器和声明对象以及与CSS语法对应的其他对象。例如：

```css
p, div {
    margin-top: 3px;
}
.error {
    color: red;
}
```

&emsp;&emsp;根据css语法解析规则，以上代码将会解析成以下对象结构</br>

```js
CSSStyleSheet: {
    CSSRule: {
        Selectors: ['p', 'div'],
        Declaration: ['margin-top', '3px']
    },
    CSSRule: {
        Selectors: ['p', 'div'],
        Declaration: ['margin-top', '3px']
    }
}
```

## 5. 浏览器解析js

* H5标准新增加了选项，可以将脚本标记为异步，此时浏览器在遇到script标签时候会用另一个线程解析执行。
* webkit和ff都对脚本的执行进行了优化，脚本执行时，另一个线程会解析文档剩余部分，找出其中需要请求的资源并加载，推测解析器不会修改DOM，而仅仅是找出了对外部资源的引用然后发起请求。（因为脚本的执行可能会改变DOM节点）
* 样式表的加载不会影响文档解析，但是，会影响到脚本的执行，所以，浏览器在加载完成样式表前，会阻止脚本的执行，这也是css建议放在头部的原因。

## 6. 渲染树

* 渲染树是在构建Dom树时候，同时构建的；
* 渲染树是文档的直观表现，该树的目的是以正确的顺序绘制内容；
* 每个渲染器代表一个矩形区域，包含高度、宽度、位置等信息；
* 矩形框类型受到该节点或者与节点相关的"display"属性影响；
* 渲染树和Dom元素不是一一对应的，非可视Dom元素不会插入渲染树(如"head"元素，"display: none"的元素同样不插入渲染树)；
* 某些Dom元素具有多个可视对象，通常这些元素都是结构比较复杂的，例如"select"元素有3个渲染器 - 一个用于显示区域，一个用于显示下拉列表框，一个用于按钮；
* 某些渲染对象对应Dom节点元素，但是不在树中相同位置，浮动和绝对定位的元素不在文档中；

## 7. style计算

* 通过计算每个元素的style属性，得到每个渲染对象的可视属性，样式属性的来源包括样式表、内联样式元素和HTML中的视觉属性*（h5标准已经废弃使用）；
* 样式计算的困难
  * 在于样式数据的庞大，众多的样式属性可能会带来内存问题；
  * 在于查找每个元素的匹配规则可能会导致性能问题；
  * 在于样式的级联规则比较复杂；
* 困难的解决方案
  * 对于样式对象的引用，webkit在某些情况下会在多个dom节点间共享这些对象，以此节省内存空间；
    * 元素必须处于相同的鼠标状态（例如，一个不能处于：悬停而另一个不是）
    * 这两个元素都不应该有id
    * 标签名称应匹配
    * 类属性应该匹配
    * 映射属性集必须相同
    * 链接状态必须匹配
    * 焦点状态必须匹配
    * 这两个元素都不应受属性选择器的影响，其中protected被定义为具有任何选择器匹配，在选择器中的任何位置都使用属性选择器
    * 元素上必须没有内联样式属性
    * 必须没有使用兄弟选择器。当遇到任何兄弟选择器时，WebCore会抛出一个全局开关，并在整个文档存在时禁用它们的样式共享。这包括+选择器和选择器，如：first-child和：last-child。
  
  * 使用规则树计算样式内容
    * 规则树，既是记录了样式属性的结构树；
    * 样式上下文可以分割为多个结构，结构体包含了特定类别（border/color等类别）的样式信息；
    * 结构的属性都是继承或者非继承的，继承属性如果未由元素定义，则继承其父类属性，非继承属性未定义，则使用默认值；
    * 规则树通过缓存整个结构，提升样式计算的速度；
    * 计算某个元素的样式上下文时，首先计算规则树中的对应路径，或者使用已有路径，然后使用该路径下的规则，在新的样式上下文中填充样式；
    * 假设存在这样的一个html结构

    ```html
    <html>
    <body>
        <div class="err" id="div1">
            <p>
                <span class="big">test1</span>
                <span class="big">test2</span>
            </p>
        </div>
        <div class="err" id="div2">another error</div>
    </body>
    </html>
    ```

    * 其对应的css结构如下

    ```css
    div {
        margin: 5px;
        color: black;
    }
    .err {
        color: red;
    }
    .big {
        margin-top: 3px;
    }
    div span {
        margin-bottom: 4px;
    }
    #div1 {
        color: blue;
    }
    #div2 {
        color: green;
    }
    ```

    * 形成的规则树和上下文树如下

    ```js
    // 规则树
    A_null = {
        B_1: {      // margin: 5px;color: black;
            C_2: {      // color: red;
                D_5: {},    // color: blue;
                F_6: {}     // color: green;
            }
        },
        E_4: {      // margin-bottom: 4px;
            G_3: {}
        }
    }

    // 上下文树
    HTML_A = {
        body_A: {
            div_D: {
                paragraph_A: {
                    span_G: {},
                    span_G: {}
                }
            },
            div_F: {}
        }
    }
    ```

    * 假设此时解析html遇到了第二个div标记，经过规则匹配，发现第1，2，6条可用，此时只需要再在上下文树新加一个节点匹配第6条规则（div_F节点）;
    * 放入上下文树的节点后，新的节点将指向规则树中对应的节点；
    * 接下来是填充样式结构，margin和color结构，因为最后的F节点没有margin结构，所以上溯规则树，直到找到先前节点插入计算过的缓存结构，即B_1上找到该缓存结构；
    * 而color由于本身F节点有了color的定义，所以无需回溯；

## 8. layout(布局)

* 计算渲染节点的位置和大小，称为布局;
* html基于流进行布局，从上到下，从左到右贯穿文档；
* 坐标系相对于根框架，包含左侧和顶部坐标；
* 根渲染节点的位置为0,0，尺寸为浏览器窗口可见部分--视口；
* 浏览器使用'dirty bit'（脏位系统）进行布局，更改或添加渲染器将自身及其子项标记为'dirty'，表示需要布局，目的在于避免对于小变化进行所有的布局重排；
* 当渲染器被标记为'dirty'时，会触发'增量布局'，这些'dirty'的渲染器将会被重新布局，如果再加上一个计时器--webkit是遍历树，在特定时间间隔后统一触发，批量执行；
* 有时候会触发全局的布局，如字体大小更改，屏幕调整大小等；
* 布局过程
  * 父渲染器决定自己宽度；
  * 父渲染器放置子渲染器，子渲染器进行自身或者子子渲染器的布局计算，最后父渲染器根据子的累计高度设置自己的height，然后供父父渲染器使用；
  * 修改'dirty'为false;