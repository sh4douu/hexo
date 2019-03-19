---
title: XSS学习系列Chapter 1：浏览器解析HTML文档
date: 2019-03-07 11:59:02
tags: 
	- XSS
	- 编码与解码
	- HTML文档解析
categories: WEB漏洞学习
---

## 一、前端基本编码知识

### 0x00 为什么要进行编码?

> 主要是因为某些数据不适合传输。原因多种多样，如Size过大，包含隐私数据，另外重要的一点就是有些字符会**引起歧义**。

对于URL，

> `&`用于分割多个参数，倘若有某个参数键值对为`name=v&lue`，就会因为`name`参数的值`v&lue`中携带了`&`而造成歧义。因此需要对`&`进行URL编码。

对于HTML，

> 当浏览器遇到`<`会识别为元素定义的开始，`>`会识别为元素的结束。倘若有`<div  id="1>" ></div>`，由于标签的属性值携带了`>`，同样会造成歧义。因此需要属性值的`>`需要进行HTML编码，即使用字符实体。

<!-- more -->

### 0x01 URL编码

> RFC3986文档规定，Url中只允许包含英文字母（a-zA-Z）、数字（0-9）、-_.~4个特殊字符以及所有保留字符。

编码方式，

> `%`加`字符在ASCII码表中的十六进制值`。
>
> 例如，`/`在ASCII码表中十六进制为`0x2f`，那么它对应的URL编码为`%2f`。

JavaScript中提供了3个函数用来对Url编码以得到合法的Url：

- escape()
- encodeURI（）
- encodeURIComponent（）

参考连接：https://www.cnblogs.com/jerrysion/p/5522673.html

### 0x02 HTML编码（字符实体）

字符实体是一个预先定义好的转义序列。

字符实体两种表示方法:

> - 字符实体以`&`开头+预先定义的`实体名称`+`;`分号结束，如“<”的实体名称为`&lt;`
> - 字符实体还可以以`&`开头+`#`符号+`字符在ASCII对应的十进制数字`+`;`分号结束，如`<`的实体编号为`&#60;`。

**字符都是有实体编号的，但有些字符是没有实体名称的。**

![](0x00-XSS学习系列之解析HTML文档\QQ截图20190307120011.png)

### 0x03 JavaScript编码

最常用的如`\uXXXX`这种写法的Unicode转义序列，表示一个字符，其中`xxxx`表示一个16进制数字，如`<`Unicode编码为`\u003c`。

## 二、解析HTML文档

解析一个HTML文档涉及三个主要过程：HTML解析——>URL解析——JavaScript解析。

每一个过程由相应的解析器进行解码解析：HTML解析器、URL解析器、JavaScript解析器。

### 0x00 HTML解析器

HTML解析器以状态机的方式运行，它从文档输入流中消耗字符并根据其转换规则转换到不同的状态。

> 当HTML解析器遇到`<`字符，且字符后不包含`/`,即不是闭合标签时，状态机就会进入**`标签打开状态（Tag Open State）`**，随后再进入**`标签名称状态（Tag Name State）`**、**`属性名称前状态（Before Attribute Name State）`**......最后进入**`数据状态（Data State）`**并且发布当前**`标签令牌（Token）`**。状态机处于**`数据状态（Data State）`**时，会继续上述的步骤，遇到完整的标签就发出**`标签令牌（Token）`**。

HTML解析器处于**`数据状态（Data State）`**、**`RCDATA 状态（RCDATA State）`**、**`属性值状态（Attribute Value State）`**时，**字符实体**会被解码为对应的字符。

例子，

> **`<div>&#60;img src=x onerror=alert(4)&#62;</div>`**
>
> `<`和`>`被编码为字符实体`&#60;`和`&#62;`。
>
> 当HTML解析器解析完`<div>`时，会进入**`数据状态（Data State）`**并发布标签令牌。接着解析到实体`&#60;`时因为处在**`数据状态（Data State）`**就会对实体进行解码为`<`，后面的`&#62;`同样道理被解码为`>`。

这里会有个问题，被解码后，`img`是否会被解析为HTML标签而导致JS执行呢？

> 答案是否定的。因为解析器在使用字符引用后不会转换到**`标签打开状态（Tag Open State）`**，不进入标签打开状态就不会被发布为HTML标签。因此，不会创建新HTML标签，只会将其作为数据来处理。
>
> **这也是为什么我们可以使用字符实体来避免用户不安全输入导致XSS的原因。**

**RCDATA**

在HTML中，属于**`RCDATA Elements`**的标签有两个：`textarea`、`title`。

**`RCDATA Elements`**类型的标签可以包含文本内容和字符实体。

解析器解析到`textarea`、`title`标签的数据部分时，状态会进入**`RCDATA State`**。

前面我们提到，处于**`RCDATA State`**状态时，字符实体是会被解析器解码的。

例子， 

> **`<textarea>&#60;script&#62;alert(5)&#60;/script&#62;</textarea>`**
>
> `<`和`>`被编码为实体`&#60;`和`&#62;`。
>
> 解析器解析到它们时会进行解码，最终得到**`<textarea><script>alert(5)</script></textarea>`**。但是里面的JS同样还是不会被执行，原因还是因为解码字符实体状态机不会进入**`标签打开状态（Tag Open State）`**，因此里面的`<script>`并不会被解析为HTML标签。

事实上，任何`RCDATA`（`textarea`、`title`里面的数据）都不会使得状态机进入**`标签打开状态（Tag Open State）`**，So if a user input wants to escape out of the CDATA context, it has to use the exact "]]>" sequence without any encoding。

HTML解析规范：https://html.spec.whatwg.org/multipage/parsing.html#tokenization

### 0x01 URL解析器

URL解析器也被建模为状态机，文档输入流中的字符可以将其导向不同的状态。

首先，要注意的是URL的`Scheme`部分（协议部分）必须为ASCII字符，即不能被任何编码，否则URL解析器的状态机将进入**`No Scheme`**状态。

例如，

> **` <a href="%6a%61%76%61%73%63%72%69%70%74:%61%6c%65%72%74%28%31%29"></a>`**
>
> URL编码部分的是`javascript:alert(1)`。
>
> JS不会被执行，因为作为`Scheme`部分的`"javascript"`这个字符串被编码，导致URL解析器状态机进入**`No Scheme`**状态。

同样，URL中的`:`也不能被以任何方式编码，否则URL解析器的状态机也将进入**`No Scheme`**状态。

例如，

> **`<a href="javascript%3aalert(3)"></a>`**
>
> 由于`:`被URL编码为`%3a`，导致URL状态机进入**`No Scheme`**状态，JS代码不能执行。

另一个例子：

**`<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;:%61%6c%65%72%74%28%32%29">`**

`"javascript"`这个字符串被实体化编码，`:`没有被编码,`alert(2)`被URL编码。

例子中的JS被成功执行。那么将产生一个问题。为什么作为URL的`Scheme`部分的`"javascript"`这个字符串被编码了但JS还是被执行了？

> 原因在于，首先，在HTML解析器中我们谈到过，HTML状态机处于**`属性值状态（Attribute Value State）`**时，字符实体时会被解码的，此处在`href`属性中，所以被实体化编码的`"javascript"`字符串会被解码。其次，HTML解析是在URL解析之前的，所以在进行URL解析之前，`Scheme`部分的`"javascript"`字符串已被解码，而并不再是被实体编码的状态。

URL解析规范：https://url.spec.whatwg.org/

URL地址结构：https://blog.csdn.net/x_nirvana/article/details/50768906

### 0x02 JavaScript解析器

JavaScript解析与HTML解析的区别在于JavaScript是上下文自由的。

在HTML中，属于**`Raw text elements`**的标签有两个：`script`、`style`。

在**`Raw text elements`**类型标签下的所有内容块都属于该标签。

存在一条特性：

> 即**`Raw text elements`**类型标签下的所有字符实体编码都不会被HTML解码。HTML解析器解析到`script`、`style`标签的内容块（数据）部分时，状态会进入**`Script Data State`**，该状态并不在我们前面说的会解码字符实体的三条状态之中。

因此，**`<script>&#97;&#108;&#101;&#114;&#116&#40;&#57;&#41;&#59</script>`**这样字符实体并不会被解码，也就不会执行JS。

形如 `\uXXXX`这样的Unicode字符转义序列或Hex编码是否能被解码需要看情况。

首先，JavaScript中有三个地方可以出现Unicode字符转义序列：

字符串中（in String）

> `Unicode escape sequences will NEVER break out of the string context in JavaScript because they will always be interpreted as string literals.`
>
> 即，Unicode转义序列出现在字符串中时，它只会被解释为普通字符，而不会破坏字符串的上下文。例如`\u000A`在Java字符串中会被解释为行终止符，会导致字符串上下文断裂。但在JavaScript中的字符串只会被解释为普通字符。简单说就是会被解码，但只解释为字符串的一部分。

标识符中（in identifier names）

> 若Unicode转义序列存在于标识符中，即变量名（如函数名等...），它会被进行解码。
>
> 例如，**`<script>\u0061\u006c\u0065\u0072\u0074(10);</script>`**
>
> 被编码转义的部分为`alert`字符，是函数名，属于在标识符中的情况，因此会被正常解码，JS代码也会被执行。

控制字符中（in control characters）

> 若Unicode转义序列存在于控制字符中，那么它会被解码但不会被解释为控制字符，而会被解释为标识符或字符串字符的一部分。
>
> 控制字符即`'`、`"`、`()`等。
>
> 例如，**`<script> alert\u0028"xss"); </script>`**，`(`进行了Unicode编码，那么解码后它不再是作为控制字符，而是作为标识符的一部分`alert(`。
>
> 因此函数的括号之类的控制字符进行Unicode转义后是不能被正常解码解释的。

总结，Unicode序列只有出现在标识符中时，才能被正常的解码解释。

例子，

> **`<script>\u0061\u006c\u0065\u0072\u0074\u0028\u0031\u0031\u0029</script>`**
>
> 被编码部分为`alert(11)`。
>
> 该例子中的JS不会被执行，因为控制字符被编码了。

> **` <script>\u0061\u006c\u0065\u0072\u0074(\u0031\u0032)</script>`**
>
> 被编码部分为`alert`及括号内为`12`。
>
> 该例子中JS不会被执行，原因在于括号内被编码的部分不能被正常解码解释，要么使用ASCII数字，要么加`""`或`''`使其变为字符串，作为字符串也只能作为普通字符。

> **`<script>alert('13\u0027)</script>`**
>
> 被编码处为`'`。
>
> 该例的JS不会执行，因为控制字符被编码了，解码后的`'`将变为字符串的一部分，而不再解释为控制字符。因此该例中字符串是不完整的，因为没有`'`来结束字符串。

> **`<script>alert('14\u000a')</script>`**
>
> 该例的JS会被执行，因为被编码的部分处于字符串内，只会被解释为普通字符，不会突破字符串上下文。

### 0x03 解析顺序

首先浏览器接收到一个HTML文档时，会触发HTML解析器对HTML文档进行词法解析，这一过程完成HTML解码并创建DOM树，接下来JavaScript解析器会介入对内联脚本进行解析，这一过程完成JS的解码工作，如果浏览器遇到需要URL的上下文环境，这时URL解析器也会介入完成URL的解码工作，URL解析器的解码顺序会根据URL所在位置不同，可能在JavaScript解析器之前或之后解析。HTML解析总是第一步。

URL解析和JavaScript解析，它们的解析顺序要根据情况而定。

例子，

> **`<a href="UserInput"></a>`**
>
> 该例子中，首先由HTML解析器对`UserInput`部分进行字符实体解码；
>
> 接着URL解析器对`UserInput`进行URL decode；
>
> 如果URL的Scheme部分为`javascript`的话，JavaScript解析器会再对`UserInput`进行解码。
>
> 所以解析顺序是：HTML解析——>URL解析——>JavaScript解析。

> **`<a href=# onclick="window.open('UserInput')"></a>`**
>
> 该例子中，首先由HTML解析器对`UserInput`部分进行字符实体解码；
>
> 接着由JavaScript解析器会再对`onclick`部分的JS进行解析并执行JS；
>
> 执行JS后`window.open('UserInput')`函数的参数会传入URL，所以再由URL解析器对`UserInput`部分进行解码。
>
> 因此解析顺序为：HTML解析——>JavaScript解析——>URL解析。

> **`<a href="javascript:window.open('UserInput')">`**
>
> 该例子中，首先还是由HTML解析器对`UserInput`部分进行字符实体解码；
>
> 接着由URL解析器解析`href`的属性值；
>
> 然后由于Scheme为`javascript`，所以由JavaScript解析；
>
> 解析执行JS后`window.open('UserInput')`函数传入URL，所以再由URL解析器解析。
>
> 所以解析顺序为：HTML解析——>URL解析——>JavaScript解析——>URL解析。

综合实例：

**`<a  href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3a;&#x25;
&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;
&#x36;&#x25;&#x33;&#x31;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;
&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x36;&#x33;&#x25;&#x35;&#x63;&#x25;
&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;
&#x35;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;
&#x25;&#x33;&#x37;&#x25;&#x33;&#x32;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;
&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x34;&#x28;&#x31;
&#x35;&#x29;"> </a>`**

首先HTML解析器进行解析，解析到`href`属性的值时，状态机进入**`属性值状态（Attribute Value State）`**，该状态会解码字符实体，解码得到结果：

![](0x00-XSS学习系列之解析HTML文档\QQ截图20190308105040.png)

接着由URL解析器进行解析并解码：

![](0x00-XSS学习系列之解析HTML文档\QQ截图20190308105254.png)

再接着由于Scheme为javascript，因此由JavaScript解析器解析并解码，加上编码部分是函数名，属于标识符，因此可以正常解码解释：

![](0x00-XSS学习系列之解析HTML文档\QQ截图20190308105533.png)

经过三轮解析解码后得到结果：`<a href="javascript:alert(15)"></a>`

参考连接：

https://www.attacker-domain.com/2013/04/deep-dive-into-browser-parsing-and-xss.html

https://security.yirendai.com/news/share/26

https://xz.aliyun.com/t/1556

https://github.com/JnuSimba/MiscSecNotes/blob/master/%E8%B7%A8%E7%AB%99%E8%84%9A%E6%9C%AC/%E8%A7%A3%E7%A0%81%E9%A1%BA%E5%BA%8F.md