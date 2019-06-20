# (二)WEB前端技术-HTML表单与CSS

<extoc></extoc>

##  HTML标签：表单标签
### 概念：用于采集用户输入的数据的。用于和服务器进行交互。

### 定义表单
html中可以使用`<form></form>` 定义表单,可以定义一个范围，范围代表采集用户数据的范围,常用属性如下

| 属性名称        | 属性描述     |
| :------------- | :------------- |
| action         | 指定提交数据的URL      |
|  method        | 指定表单提交方式,`GET`/`POST` |

### 表单提交方式

提交方式一共有7种,其中2种比较常用

```
* get：
    1. 请求参数会在地址栏中显示。会封装到请求行中(HTTP协议后讲解)。
    2. 请求参数大小是有限制的。
    3. 不太安全。
* post：
    2. 请求参数不会再地址栏中显示。会封装在请求体中(HTTP协议后讲解)
    2. 请求参数的大小没有限制。
    3. 较为安全。
* 表单项中的数据要想被提交：必须指定其name属性
```




### 表单项标签

#### `<input>`标签:输入框标签,可以通过type属性值，改变元素展示的样式
##### 属性
| 属性名称     | 属性介绍     |
| :------------- | :------------- |
| name           | 表单项的名称(提交数据必须)    |
| type           | text：文本输入框，默认值 <br>  password：密码输入框 <br>   radio:单选框  <br>   checkbox：复选框 <br>  file：文件选择框  <br>   hidden：隐藏域，用于提交一些信息  <br> submit：提交按钮。可以提交表单 <br> button：普通按钮  <br>  |
| placeholder   | 如果是`type=input`,指定输入框的提示信息，当输入框的内容发生变化，会自动清空提示信息  |

##### 注意
  **如果`type=text`**
    1. 可以使用placeholder指定输入框的提示信息，当输入框的内容发生变化，会自动清空提示信息

  **如果`type=radio`**
    1. 要想让多个单选框实现单选的效果，则多个单选框的name属性值必须一样。
    2. 一般会给每一个单选框提供value属性，指定其被选中后提交的值
    3. checked属性，可以指定默认值

  **如果`type= checkbox`**
    1. 一般会给每一个单选框提供value属性，指定其被选中后提交的值
    2. checked属性，可以指定默认值

  **可以使用`<label>`标签指定输入项的文字描述信息**
    1. label的for属性一般会和 input 的 id属性值 对应。如果对应了，则点击label区域，会让input输入框获取焦点。

##### 示例
```html
<label for="username"> 用户名 </label>：<input type="text" name="username" placeholder="请输入用户名" id="username"><br>

密码：<input type="password" name="password" placeholder="请输入密码"><br>

性别：<input type="radio" name="gender" value="male" >  男
     <input type="radio" name="gender" value="female" checked>  女
    <br>
爱好：<input type="checkbox" name="hobby" value="shopping"> 逛街
    <input type="checkbox" name="hobby" value="java"  checked> Java
    <input type="checkbox" name="hobby" value="game"> 游戏<br>

图片：<input type="file" name="file"><br>

隐藏域：<input type="hidden" name="id" value="aaa"> <br>
<!-- html5新加入的 -->
取色器：<input type="color" name="color"><br>
生日：<input type="date" name="birthday"> <br>
生日：<input type="datetime-local" name="birthday"> <br>
邮箱：<input type="email" name="email"> <br>
年龄：<input type="number" name="age"> <br>
```

#### `<select>`标签:下拉列表,使用option，指定列表项
```html
<select name="province">
  <option value="">--请选择--</option>
  <option value="1">北京</option>
  <option value="2">上海</option>
  <option value="3" selected>陕西</option>
</select><br>
```

#### `<textarea>`标签:文本域
```html
<textarea cols="20" rows="5" name="des"></textarea>
```

### 案例:注册页面
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
</head>
<body>
    <!--定义表单 form-->
    <form action="#" method="post">
        <table border="1" align="center" width="500">
            <tr>
                <td><label for="username">用户名</label></td>
                <td><input type="text" name="username" id="username"></td>
            </tr>

            <tr>
                <td><label for="password">密码</label></td>
                <td><input type="password" name="password" id="password"></td>
            </tr>

            <tr>
                <td><label for="email">Email</label></td>
                <td><input type="email" name="email" id="email"></td>
            </tr>

            <tr>
                <td><label for="name">姓名</label></td>
                <td><input type="text" name="name" id="name"></td>
            </tr>

            <tr>
                <td><label for="tel">手机号</label></td>
                <td><input type="text" name="tel" id="tel"></td>
            </tr>

            <tr>
                <td><label>性别</label></td>
                <td><input type="radio" name="gender" value="male"> 男
                    <input type="radio" name="gender" value="female"> 女
                </td>
            </tr>

            <tr>
                <td><label for="birthday">出生日期</label></td>
                <td><input type="date" name="birthday" id="birthday"></td>
            </tr>

            <tr>
                <td><label for="checkcode">验证码</label></td>
                <td><input type="text" name="checkcode" id="checkcode">
                    <img src="img/verify_code.jpg">
                </td>
            </tr>


            <tr>
                <td colspan="2" align="center"><input type="submit" value="注册"></td>
            </tr>
        </table>
    </form>
</body>
</html>
```


## CSS：页面美化和布局控制
### 概念
    CSS: Cascading Style Sheets 层叠样式表

    层叠：多个样式可以作用在同一个html的元素上，同时生效

    样式表 : 通过一系列样式调整网页展示效果

#### 好处
```
1. 功能强大
2. 将内容展示和样式控制分离
	* 降低耦合度。解耦
	* 让分工协作更容易
	* 提高开发效率
```

### CSS的使用：CSS与html结合方式(掌握)

#### 内联样式
```
* 在标签内使用style属性指定css代码
* 如：<div style="color:red;">hello css</div>
```
#### 内部样式
```
* 在head标签内，定义style标签，style标签的标签体内容就是css代码
* 如：
  <style>
        div{
            color:blue;
        }

    </style>
  <div>hello css</div>
```
#### 外部样式
```
1. 定义css资源文件。
2. 在head标签内，定义link标签，引入外部的资源文件
* 如：
  * a.css文件：
  div{
      color:green;
  }
<link rel="stylesheet" href="css/a.css">
<div>hello css</div>
<div>hello css</div>
```

#### 注意
```
1. 1,2,3种方式 css作用范围越来越大
2. 1方式不常用，后期常用2,3
3. 3种格式可以写为：
  <style>
        @import "css/a.css";
  </style>
```

### CSS语法
```css
选择器 {
  属性名1:属性值1;
  属性名2:属性值2;
  ...
}
```
选择器:筛选具有相似特征的元素

**注意：每一对属性需要使用`;`隔开，最后一对属性可以不加**


### 选择器：筛选具有相似特征的元素
#### 基础选择器
```
1. id选择器：选择具体的id属性值的元素.建议在一个html页面中id值唯一
    * 语法：#id属性值{}
2. 元素选择器：选择具有相同标签名称的元素
    * 语法： 标签名称{}
    * 注意：id选择器优先级高于元素选择器
3. 类选择器：选择具有相同的class属性值的元素。
    * 语法：.class属性值{}
    * 注意：类选择器选择器优先级高于元素选择器
```

##### 示例
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>基础选择器</title>
    <style>
        .cls1{

            color: blue;
        }

        div{
            color:green;
        }

        #div1{
               color: red;
        }
    </style>
</head>
<body>
  <div id="div1">传智播客</div>
  <div class="cls1">黑马程序员</div>

  <p class="cls1">传智学院</p>

</body>
</html>
```

#### 扩展选择器
```
1. 选择所有元素：
  * 语法： *{}
2. 并集选择器：
  * 选择器1,选择器2{}

3. 后代元素选择器：选择选择器1下的所有符合选择器2的标签元素 不论层级
  * 语法：  选择器1 选择器2{}
4. 子元素选择器：选择选择器1下符合选择器2的第一层级的后代元素   只选择第一层级
  * 语法：  选择器1 > 选择器2{}

5. 属性选择器：选择元素名称，属性名=属性值的元素
  * 语法：  元素名称[属性名="属性值"]{}

6. 伪类选择器：选择一些元素具有的状态
  * 语法： 元素:状态{}
  * 如： <a>
    * 状态：
      * link：初始化的状态
      * visited：被访问过的状态
      * active：正在访问状态
      * hover：鼠标悬浮状态
```

##### 示例
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>扩展选择器</title>

    <style>

        /* 后代元素选择器:  选择选择器1中的所有符合选择器2的元素   不论层级   */
         div span{
                color: red;
         }
        /* 子元素选择器  : 选择的是选择器1中第一次层级的后代元素   儿子  */
        div > span{
            color: red;
            border: 1px solid red;
        }


        /*属性选择器:选择指定属性名称等于指定值的标签元素*/
        input[type='text'] {
            border: 5px solid;
        }

        /*伪类选择器:    :link  链接状态   :hover 鼠标悬浮状态   :active  鼠标按下状态  :visited 访问状态   */
        a:link {
            color: pink;
        }

        a:hover{
            color: green;
        }

        a:active{
            color: yellow;
        }

        a:visited{
            color: red;
        }

    </style>
</head>
<body>

    <p>
        <span>传智播客</span>
    </p>
    <span>黑马程序员</span>

    <input type="text" >
    <input type="password" >

    <br>    <br>    <br>

    <a href="#">黑马程序员</a>
</body>
</html>
```

### 样式属性
#### 字体、文本
```
font-size：10px ;       /*字体大小*/
color：#00FF00 ;        /*文本颜色,色值或者英文单词*/
text-align：left ;     /*对其方式*/
line-height：100px;    /*行高,可以让文本垂直居中*/
```

#### 背景
```
background-color：red ; /*背景颜色,色值或者英文单词*/
background-image:url(imgs/1.jpg) ;   /*背景颜图片,固定写法,url中图片路径*/
```

#### 边框
```
border：1px solid red ;  /*设置四周边框宽度为1px,样式为实线,颜色为红色*/
```

#### 尺寸
```
width：100px ;           /*宽度*/
height：100px ;         /*高度*/
```

#### 盒子模型：控制布局
```
margin：10px ;  /*外边距*/
padding：10px ; /*内边距*/
/*注意:padding会影响盒子最终的大小,可以使用下面的属性进行设置*/
box-sizing: border-box;  /*设置盒子的属性，让width和height就是最终盒子的大小*/
```

#### float：浮动
```
float: left ;   /*左浮动*/
float:right ;   /*右边距*/
clear:both  ;   /*清除浮动*/
```

## 案例：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
<style>
    *{
        margin: 0px;
        padding: 0px;
        box-sizing: border-box;
    }
    body{
        background: url("img/register_bg.png") no-repeat center;
        padding-top: 25px;
    }

    .rg_layout{
        width: 900px;
        height: 500px;
        border: 8px solid #EEEEEE;
        background-color: white;
        /*让div水平居中*/
        margin: auto;
    }

    .rg_left{
        /*border: 1px solid red;*/
        float: left;
        margin: 15px;
    }
    .rg_left > p:first-child{
        color:#FFD026;
        font-size: 20px;
    }

    .rg_left > p:last-child{
        color:#A6A6A6;
        font-size: 20px;

    }


    .rg_center{
        float: left;
       /* border: 1px solid red;*/

    }

    .rg_right{
        /*border: 1px solid red;*/
        float: right;
        margin: 15px;
    }

    .rg_right > p:first-child{
        font-size: 15px;

    }
    .rg_right p a {
        color:pink;
    }

    .td_left{
        width: 100px;
        text-align: right;
        height: 45px;
    }
    .td_right{
        padding-left: 50px ;
    }

    #username,#password,#email,#name,#tel,#birthday,#checkcode{
        width: 251px;
        height: 32px;
        border: 1px solid #A6A6A6 ;
        /*设置边框圆角*/
        border-radius: 5px;
        padding-left: 10px;
    }
    #checkcode{
        width: 110px;
    }

    #img_check{
        height: 32px;
        vertical-align: middle;
    }

    #btn_sub{
        width: 150px;
        height: 40px;
        background-color: #FFD026;
        border: 1px solid #FFD026 ;
    }

</style>
</head>
<body>
<div class="rg_layout">
    <div class="rg_left">
        <p>新用户注册</p>
        <p>USER REGISTER</p>

    </div>

    <div class="rg_center">
        <div class="rg_form">
            <!--定义表单 form-->
            <form action="#" method="post">
                <table>
                    <tr>
                        <td class="td_left"><label for="username">用户名</label></td>
                        <td class="td_right"><input type="text" name="username" id="username" placeholder="请输入用户名"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="password">密码</label></td>
                        <td class="td_right"><input type="password" name="password" id="password" placeholder="请输入密码"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="email">Email</label></td>
                        <td class="td_right"><input type="email" name="email" id="email" placeholder="请输入邮箱"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="name">姓名</label></td>
                        <td class="td_right"><input type="text" name="name" id="name" placeholder="请输入姓名"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="tel">手机号</label></td>
                        <td class="td_right"><input type="text" name="tel" id="tel" placeholder="请输入手机号"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label>性别</label></td>
                        <td class="td_right">
                            <input type="radio" name="gender" value="male"> 男
                            <input type="radio" name="gender" value="female"> 女
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="birthday">出生日期</label></td>
                        <td class="td_right"><input type="date" name="birthday" id="birthday" placeholder="请输入出生日期"></td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="checkcode" >验证码</label></td>
                        <td class="td_right"><input type="text" name="checkcode" id="checkcode" placeholder="请输入验证码">
                            <img id="img_check" src="img/verify_code.jpg">
                        </td>
                    </tr>
                    <tr>
                        <td colspan="2" align="center"><input type="submit" id="btn_sub" value="注册"></td>
                    </tr>
                </table>
            </form>
        </div>
    </div>
    <div class="rg_right">
        <p>已有账号?<a href="#">立即登录</a></p>
    </div>
</div>
</body>
</html>
```
