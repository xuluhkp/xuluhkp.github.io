# 今日内容介绍

<extoc></extoc>

# JavaScript - DOM
## HTML DOM
### 标签体的设置和获取:`innerHTML`(掌握)
    1. Element对象的innerHTML属性可以获取和设置元素的内容体
    2. `var html = element.innerHTML`  获取元素的内容体
    3. `element.innerHTML = "新内容"` 设置元素的内容体

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="div1">传智博客....</div>
<script>
    //获取元素内容体
    var html = document.getElementById("div1").innerHTML;
    alert(html);

    // 设置元素内容体
    document.getElementById("div1").innerHTML = "<font color='red'>黑马程序员...</font>" ;
</script>
</body>
</html>
```

#### 改写动态表格
```JavaScript
//使用innerHTML添加
 document.getElementById("btn_add").onclick = function() {
     //2.获取文本框的内容
     var id = document.getElementById("id").value;
     var name = document.getElementById("name").value;
     var gender = document.getElementById("gender").value;

     //获取table
     var table = document.getElementsByTagName("table")[0];

     //追加一行
     table.innerHTML += "<tr> <td>"+id+"</td>  <td>"+name+"</td> <td>"+gender+"</td> <td><a href=\"javascript:void(0);\" onclick=\"delTr(this);\" >删除</a></td> </tr>";
 }
```

### 操作html元素对象的属性(掌握)
#### 步骤
    1. 确定需要获取的标签
    2. 修改元素属性

### 控制元素样式(掌握)
#### 使用元素的style属性来设置 `element.style.样式属性 = 属性值`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div{
            width: 100px;
            height: 100px;
            background-color: red;
        }
    </style>
</head>
<body>
<div id="div1">

</div>
<script>
    //获取需要操作的元素对象
    var ele = document.getElementById("div1");
    //给元素对象绑定点击事件
    ele.onclick = function () {
        //用户点击触发事件,修改元素样式
        ele.style.width = "200px";
        ele.style.height = "200px";
        ele.style.backgroundColor = "skyblue";
    }
</script>
</body>
</html>
```

#### 提前定义好类选择器的样式，通过元素的className属性来设置其class属性值。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div{
            width: 100px;
            height: 100px;
            background-color: red;
        }

        .d1{
            width: 200px;
            height: 200px;
            background-color: skyblue;
        }
    </style>
</head>
<body>
<div id="div1">

</div>
<script>
    //获取需要操作的元素对象
    var ele = document.getElementById("div1");
    //给元素对象绑定点击事件
    ele.onclick = function () {
        //修改元素对象的class属性值
       ele.className = "d1";
    }
</script>
</body>
</html>
```


## 事件监听机制
### 概念:某些组件被执行了某些操作后，触发某些代码的执行。

    * 事件:某些操作。如: 单击，双击，键盘按下了，鼠标移动了
		* 事件源:组件。如: 按钮 文本输入框...
		* 监听器:代码。
		* 注册监听:将事件，事件源，监听器结合在一起。 当事件源上发生了某个事件，则触发执行某个监听器代码。


### 常见的事件(掌握)

| 事件类型 | 事件属性     |  触发时机     |
| :------------- | :------------- | :------------- |
| 点击事件       |  onclick  <br>  ondblclick  |  鼠标单击事件   <br>  鼠标双击事件 |
| 焦点事件       |  onblur  <br> onfocus       |   失去焦点  <br>   元素获得焦点    |
| 加载完毕事件   |  onload                     |  一张页面或一幅图像完成加载 |
| 鼠标事件      |   onmousedown <br> onmouseup  <br>  onmousemove  <br>   onmouseover    <br>   onmouseout|  鼠标按钮被按下  <br> 鼠标按键被松开  <br> 鼠标被移动  <br> 鼠标移到某元素之上 <br> 鼠标从某元素移开 |
| 键盘事件      | onkeydown  <br>  onkeyup  <br> onkeypress  |  某个键盘按键被按下 <br>  某个键盘按键被松开 <br> 某个键盘按键被按下并松开 |
| 选择和改变事件  | onchange <br>  onselect   |  域的内容被改变  <br>  文本被选中 |
| 表单事件  |  onsubmit  <br>  onreset | 确认按钮被点击 <br>  重置按钮被点击 |


### 示例代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>常见事件</title>
    <script>
        //2.加载完成事件  onload
        window.onload = function(){
            //1.失去焦点事件
            document.getElementById("username").onblur = function(){
                alert("失去焦点了...");
            }
            //3.绑定鼠标移动到元素之上事件
            document.getElementById("username").onmouseover = function(){
                alert("鼠标来了....");
            }

            //3.绑定鼠标点击事件
            document.getElementById("username").onmousedown = function(event){
               // alert("鼠标点击了....");
                alert(event.button);
            }

           document.getElementById("username").onkeydown = function(event){
                // alert("鼠标点击了....");
               // alert(event.button);
                if(event.keyCode == 13){
                    alert("提交表单");
                }
            }

            document.getElementById("username").onchange = function(event){
                alert("改变了...")
            }

            document.getElementById("city").onchange = function(event){
                alert("改变了...")
            }

            document.getElementById("form").onsubmit = function(){
                //校验用户名格式是否正确
                var flag = false;
                return flag;
            }
        }

        function checkForm(){
            return true;
        }


    </script>

</head>
<body>
<form action="#" id="form" onclick="return  checkForm();">
<input name="username" id="username">

<select id="city">
    <option>--请选择--</option>
    <option>北京</option>
    <option>上海</option>
    <option>西安</option>
</select>
<input type="submit" value="提交">
</form>
</body>
</html>
```

### 注意:
    1. 表单提交事件应该绑定在form标签身上
    2. 注意事件函数的位置,不要写在其他函数中,包括onload事件
    3. onsubmit事件可以控制表单是否提交 ,事件函数中`return  false`阻止表单提交
    4. 特别需要注意,如果是在form标签身上写属性的方式绑定事件,需要在函数调用之前添加return


### 案例五:表格的全选和全不选(掌握)
#### 需求
    1. 需求一:用户点击全选按钮,所有行全部选中
    2. 需求二:用户点击全不选按钮,所有行全部取消选中
    3. 需求三:用户点击反选按钮,所有行选中的变为不选中,不选中的变为选中
    4. 需求四:用户点击表头的复选框,后面所有的复选框的状态和表头的复选框一致
    5. 需求五:用户鼠标停在哪一行,哪一行变色

#### 示例代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表格全选</title>
    <style>
        table{
            border: 1px solid;
            width: 500px;
            margin-left: 30%;
        }

        td,th{
            text-align: center;
            border: 1px solid;
        }
        div{
            margin-top: 10px;
            margin-left: 30%;
        }

        .over{
            background-color: pink;
        }
        .out{
            background-color: white;
        }
    </style>

    <script>

        window.onload  = function(){
            //需求一:用户点击全选按钮,所有行全部选中
            document.getElementById("selectAll").onclick = function(){
                var cbs = document.getElementsByName("cb");
                for (var i = 0; i < cbs.length; i++) {
                    cbs[i].checked = true;
                }

            }

            //需求二:用户点击全不选按钮,所有行全部取消选中
            document.getElementById("unSelectAll").onclick = function(){
                var cbs = document.getElementsByName("cb");
                for (var i = 0; i < cbs.length; i++) {
                    cbs[i].checked = false;
                }

            }

            //需求三:用户点击反选按钮,所有行选中的变为不选中,不选中的变为选中
            document.getElementById("selectRev").onclick = function(){
                var cbs = document.getElementsByName("cb");
                for (var i = 0; i < cbs.length; i++) {
                    cbs[i].checked = !cbs[i].checked
                }

            }

            //需求四:用户点击表头的复选框,后面所有的复选框的状态和表头的复选框一致
            document.getElementById("firstcb").onclick = function(){
                var cbs = document.getElementsByName("cb");
                var firstcb = document.getElementById("firstcb");
                for (var i = 0; i < cbs.length; i++) {
                    cbs[i].checked = firstcb.checked;
                }

            }

            //需求五:用户鼠标停在哪一行,哪一行变色
            var trs = document.getElementsByTagName("tr");
            for (var i = 0; i < trs.length; i++) {
                trs[i].onmouseover = function(){
                    this.className = "over"
                }

                trs[i].onmouseout = function(){
                    this.className = "out"
                }
            }
        }
    </script>
</head>
<body>

<table>
    <caption>学生信息表</caption>
    <tr>
        <th><input type="checkbox" name="cb" id="firstcb"></th>
        <th>编号</th>
        <th>姓名</th>
        <th>性别</th>
        <th>操作</th>
    </tr>

    <tr>
        <td><input type="checkbox" name="cb"></td>
        <td>1</td>
        <td>令狐冲</td>
        <td>男</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

    <tr>
        <td><input type="checkbox" name="cb"></td>
        <td>2</td>
        <td>任我行</td>
        <td>男</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

    <tr>
        <td><input type="checkbox" name="cb"></td>
        <td>3</td>
        <td>岳不群</td>
        <td>?</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

</table>
<div>
    <input type="button" id="selectAll" value="全选">
    <input type="button" id="unSelectAll" value="全不选">
    <input type="button" id="selectRev" value="反选">
</div>
</body>
</html>
```


### 案例六:简单表单校验(掌握)
#### 需求
    1. 需求一:用户输入用户名和密码离开焦点,校验用户名和密码输入的数据是否符合规则(6-12位字母和数字)
    2. 需求二:用户点击提交按钮,校验用户名和密码是否符合规则,如果符合规则提交表单,不符合规则阻止表单提交

#### 示例代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
    <style>
        * {
            margin: 0px;
            padding: 0px;
            box-sizing: border-box;
        }
        body {
            background: url("img/register_bg.png") no-repeat center;
            padding-top: 25px;
        }

        .rg_layout {
            width: 900px;
            height: 500px;
            border: 8px solid #EEEEEE;
            background-color: white;
            /*让div水平居中*/
            margin: auto;
        }

        .rg_left {
            /*border: 1px solid red;*/
            float: left;
            margin: 15px;
        }

        .rg_left > p:first-child {
            color: #FFD026;
            font-size: 20px;
        }

        .rg_left > p:last-child {
            color: #A6A6A6;
            font-size: 20px;

        }

        .rg_center {
            float: left;
            /* border: 1px solid red;*/

        }

        .rg_right {
            /*border: 1px solid red;*/
            float: right;
            margin: 15px;
        }

        .rg_right > p:first-child {
            font-size: 15px;

        }

        .rg_right p a {
            color: pink;
        }

        .td_left {
            width: 100px;
            text-align: right;
            height: 45px;
        }

        .td_right {
            padding-left: 50px;
        }

        #username, #password, #email, #name, #tel, #birthday, #checkcode {
            width: 251px;
            height: 32px;
            border: 1px solid #A6A6A6;
            /*设置边框圆角*/
            border-radius: 5px;
            padding-left: 10px;
        }

        #checkcode {
            width: 110px;
        }

        #img_check {
            height: 32px;
            vertical-align: middle;
        }

        #btn_sub {
            width: 150px;
            height: 40px;
            background-color: #FFD026;
            border: 1px solid #FFD026;
        }

        #td_sub{
            padding-left: 150px;
        }

        .error{
            color:red;
            vertical-align: middle;
        }
    </style>
<script>
    window.onload = function(){
        document.getElementById("form").onsubmit = function(){
            //验证用户名
            //验证密码
            //...
            //都成功则返回true
            //
            return checkUsername() && checkPassword();
        }

        document.getElementById("username").onblur = checkUsername;
        document.getElementById("password").onblur = checkPassword;
    }

    function checkUsername(){
        var username = document.getElementById("username").value;
        var reg_username = /^\w{6,12}$/;
        var flag = reg_username.test(username);
        var s_username = document.getElementById("s_username");
        if(flag){
            s_username.innerHTML = "<img height='25' width='35' src='img/gou.png'>"
        }else{
            s_username.innerHTML = "用户名格式有误";
        }
        return flag;
    }

    function checkPassword(){
        var password = document.getElementById("password").value;
        var reg_password = /^\w{6,12}$/;
        var flag = reg_password.test(password);
        var s_password = document.getElementById("s_password");
        if(flag){
            s_password.innerHTML = "<img height='25' width='35' src='img/gou.png'>"
        }else{
            s_password.innerHTML = "密码格式有误";
        }
        return flag;
    }

</script>
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
            <form action="#" id="form" method="get">
                <table>
                    <tr>
                        <td class="td_left"><label for="username">用户名</label></td>
                        <td class="td_right">
                            <input type="text" name="username" id="username" placeholder="请输入用户名">
                            <span id="s_username" class="error"></span>
                        </td>

                    </tr>

                    <tr>
                        <td class="td_left"><label for="password">密码</label></td>
                        <td class="td_right">
                            <input type="password" name="password" id="password" placeholder="请输入密码">
                            <span id="s_password" class="error"></span>
                        </td>
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
                        <td class="td_right"><input type="date" name="birthday" id="birthday" placeholder="请输入出生日期">
                        </td>
                    </tr>

                    <tr>
                        <td class="td_left"><label for="checkcode">验证码</label></td>
                        <td class="td_right"><input type="text" name="checkcode" id="checkcode" placeholder="请输入验证码">
                            <img id="img_check" src="img/verify_code.jpg">
                        </td>
                    </tr>


                    <tr>
                        <td colspan="2"  id="td_sub"><input type="submit" id="btn_sub" value="注册"></td>
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
# Bootstrap简介
## 概念
```
* 一个前端开发的框架，Bootstrap，来自 Twitter，是目前很受欢迎的前端框架。Bootstrap 是基于 HTML、CSS、JavaScript 的，它简洁灵活，使得 Web 开发更加快捷。
* 框架:一个半成品软件，开发人员可以在框架基础上，在进行开发，简化编码。
* 好处：
  1. 定义了很多的css样式和js插件。我们开发人员直接可以使用这些样式和插件得到丰富的页面效果。
  2. 响应式布局:同一套页面可以兼容不同分辨率的设备。
```

## 快速入门
		1. 下载Bootstrap
		2. 在项目中将这三个文件夹复制
		3. 创建html页面，引入必要的资源文件

```HTML
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>Bootstrap HelloWorld</title>

    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">


    <!-- jQuery (Bootstrap 的所有 JavaScript 插件都依赖 jQuery，所以必须放在前边) -->
    <script src="js/jquery-3.2.1.min.js"></script>
    <!-- 加载 Bootstrap 的所有 JavaScript 插件。你也可以根据需要只加载单个插件。 -->
    <script src="js/bootstrap.min.js"></script>
</head>
<body>
<h1>你好，世界！</h1>

</body>
</html>
```

## 响应式布局
  	* 同一套页面可以兼容不同分辨率的设备。
  	* 实现：依赖于栅格系统：将一行平均分成12个格子，可以指定元素占几个格子
  	* 步骤：
  		1. 定义容器。相当于之前的table、
  			* 容器分类：
  				1. container：两边留白
  				2. container-fluid：每一种设备都是100%宽度
  		2. 定义行。相当于之前的tr   样式：row
  		3. 定义元素。指定该元素在不同的设备上，所占的格子数目。样式：col-设备代号-格子数目
  			* 设备代号：
  				1. xs：超小屏幕 手机 (<768px)：col-xs-12
  				2. sm：小屏幕 平板 (≥768px)
  				3. md：中等屏幕 桌面显示器 (≥992px)
  				4. lg：大屏幕 大桌面显示器 (≥1200px)

  		* 注意：
  			1. 一行中如果格子数目超过12，则超出部分自动换行。
  			2. 栅格类属性可以向上兼容。栅格类适用于与屏幕宽度大于或等于分界点大小的设备。
  			3. 如果真实设备宽度小于了设置栅格类属性的设备代码的最小值，会一个元素沾满一整行。

## CSS样式和JS插件(详细见BootStrap官方文档)
  	1. 全局CSS样式：
  		* 按钮：class="btn btn-default"
  		* 图片：
  			*  class="img-responsive"：图片在任意尺寸都占100%
  			*  图片形状
  				*  <img src="..." alt="..." class="img-rounded">：方形
  				*  <img src="..." alt="..." class="img-circle"> ： 圆形
  				*  <img src="..." alt="..." class="img-thumbnail"> ：相框
  		* 表格
  			* table
  			* table-bordered
  			* table-hover
  		* 表单
  			* 给表单项添加：class="form-control"
  	2. 组件：
  		* 导航条
  		* 分页条
  	3. 插件：
  		* 轮播图

## 案例
```html

	<!DOCTYPE html>
	<html lang="zh-CN">
	<head>
	    <meta charset="utf-8">
	    <meta http-equiv="X-UA-Compatible" content="IE=edge">
	    <meta name="viewport" content="width=device-width, initial-scale=1">
	    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
	    <title>Bootstrap HelloWorld</title>

	    <!-- Bootstrap -->
	    <link href="css/bootstrap.min.css" rel="stylesheet">


	    <!-- jQuery (Bootstrap 的所有 JavaScript 插件都依赖 jQuery，所以必须放在前边) -->
	    <script src="js/jquery-3.2.1.min.js"></script>
	    <!-- 加载 Bootstrap 的所有 JavaScript 插件。你也可以根据需要只加载单个插件。 -->
	    <script src="js/bootstrap.min.js"></script>
	    <style>
	        .paddtop{
	            padding-top: 10px;
	        }
	        .search-btn{
	            float: left;
	            border:1px solid #ffc900;
	            width: 90px;
	            height: 35px;
	            background-color:#ffc900 ;
	            text-align: center;
	            line-height: 35px;
	            margin-top: 15px;
	        }

	        .search-input{
	            float: left;
	            border:2px solid #ffc900;
	            width: 400px;
	            height: 35px;
	            padding-left: 5px;
	            margin-top: 15px;
	        }
	        .jx{
	            border-bottom: 2px solid #ffc900;
	            padding: 5px;
	        }
	        .company{
	            height: 40px;
	            background-color: #ffc900;
	            text-align: center;
	            line-height:40px ;
	            font-size: 8px;
	        }
	    </style>
	</head>
	<body>

	   <!-- 1.页眉部分-->
	   <header class="container-fluid">
	       <div class="row">
	           <img src="img/top_banner.jpg" class="img-responsive">
	       </div>
	       <div class="row paddtop">
	           <div class="col-md-3">
	               <img src="img/logo.jpg" class="img-responsive">
	           </div>
	           <div class="col-md-5">
	               <input class="search-input" placeholder="请输入线路名称">
	               <a class="search-btn" href="#">搜索</a>
	           </div>
	           <div class="col-md-4">
	               <img src="img/hotel_tel.png" class="img-responsive">
	           </div>

	       </div>
	       <!--导航栏-->
	       <div class="row">
	           <nav class="navbar navbar-default">
	               <div class="container-fluid">
	                   <!-- Brand and toggle get grouped for better mobile display -->
	                   <div class="navbar-header">
	                       <!-- 定义汉堡按钮 -->
	                       <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
	                           <span class="sr-only">Toggle navigation</span>
	                           <span class="icon-bar"></span>
	                           <span class="icon-bar"></span>
	                           <span class="icon-bar"></span>
	                       </button>
	                       <a class="navbar-brand" href="#">首页</a>
	                   </div>

	                   <!-- Collect the nav links, forms, and other content for toggling -->
	                   <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
	                       <ul class="nav navbar-nav">
	                           <li class="active"><a href="#">Link <span class="sr-only">(current)</span></a></li>
	                           <li><a href="#">Link</a></li>
	                           <li><a href="#">Link</a></li>
	                           <li><a href="#">Link</a></li>
	                           <li><a href="#">Link</a></li>
	                           <li><a href="#">Link</a></li>

	                       </ul>
	                   </div><!-- /.navbar-collapse -->
	               </div><!-- /.container-fluid -->
	           </nav>

	       </div>

	       <!--轮播图-->
	       <div class="row">
	           <div id="carousel-example-generic" class="carousel slide" data-ride="carousel">
	               <!-- Indicators -->
	               <ol class="carousel-indicators">
	                   <li data-target="#carousel-example-generic" data-slide-to="0" class="active"></li>
	                   <li data-target="#carousel-example-generic" data-slide-to="1"></li>
	                   <li data-target="#carousel-example-generic" data-slide-to="2"></li>
	               </ol>

	               <!-- Wrapper for slides -->
	               <div class="carousel-inner" role="listbox">
	                   <div class="item active">
	                       <img src="img/banner_1.jpg" alt="...">
	                   </div>
	                   <div class="item">
	                       <img src="img/banner_2.jpg" alt="...">
	                   </div>
	                   <div class="item">
	                       <img src="img/banner_3.jpg" alt="...">
	                   </div>

	               </div>

	               <!-- Controls -->
	               <a class="left carousel-control" href="#carousel-example-generic" role="button" data-slide="prev">
	                   <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
	                   <span class="sr-only">Previous</span>
	               </a>
	               <a class="right carousel-control" href="#carousel-example-generic" role="button" data-slide="next">
	                   <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
	                   <span class="sr-only">Next</span>
	               </a>
	           </div>



	       </div>

	   </header>
	   <!-- 2.主体部分-->
	   <div class="container">
	        <div class="row jx">
	            <img src="img/icon_5.jpg">
	            <span>黑马精选</span>
	        </div>

	       <div class="row paddtop">
	           <div class="col-md-3">
	                <div class="thumbnail">
	                    <img src="img/jiangxuan_3.jpg" alt="">
	                    <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                    <font color="red">&yen; 699</font>
	                </div>
	           </div>
	           <div class="col-md-3">
	               <div class="thumbnail">
	                   <img src="img/jiangxuan_3.jpg" alt="">
	                   <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                   <font color="red">&yen; 699</font>
	               </div>

	           </div>
	           <div class="col-md-3">

	               <div class="thumbnail">
	                   <img src="img/jiangxuan_3.jpg" alt="">
	                   <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                   <font color="red">&yen; 699</font>
	               </div>
	           </div>
	           <div class="col-md-3">

	               <div class="thumbnail">
	                   <img src="img/jiangxuan_3.jpg" alt="">
	                   <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                   <font color="red">&yen; 699</font>
	               </div>
	           </div>


	       </div>
	       <div class="row jx">
	           <img src="img/icon_6.jpg">
	           <span>国内游</span>
	       </div>
	       <div class="row paddtop">
	           <div class="col-md-4">
	               <img src="img/guonei_1.jpg">
	           </div>
	           <div class="col-md-8">
	               <div class="row">
	                   <div class="col-md-4">
	                       <div class="thumbnail">
	                           <img src="img/jiangxuan_3.jpg" alt="">
	                           <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                           <font color="red">&yen; 699</font>
	                       </div>
	                   </div>
	                   <div class="col-md-4">
	                       <div class="thumbnail">
	                           <img src="img/jiangxuan_3.jpg" alt="">
	                           <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                           <font color="red">&yen; 699</font>
	                       </div>

	                   </div>
	                   <div class="col-md-4">

	                       <div class="thumbnail">
	                           <img src="img/jiangxuan_3.jpg" alt="">
	                           <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                           <font color="red">&yen; 699</font>
	                       </div>
	                   </div>

	               </div>
	               <div class="row">
	                   <div class="col-md-4">
	                       <div class="thumbnail">
	                           <img src="img/jiangxuan_3.jpg" alt="">
	                           <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                           <font color="red">&yen; 699</font>
	                       </div>
	                   </div>
	                   <div class="col-md-4">
	                       <div class="thumbnail">
	                           <img src="img/jiangxuan_3.jpg" alt="">
	                           <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                           <font color="red">&yen; 699</font>
	                       </div>

	                   </div>
	                   <div class="col-md-4">

	                       <div class="thumbnail">
	                           <img src="img/jiangxuan_3.jpg" alt="">
	                           <p>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>
	                           <font color="red">&yen; 699</font>
	                       </div>
	                   </div>


	               </div>

	           </div>

	       </div>
	   </div>
	   <!-- 3.页脚部分-->
	   <footer class="container-fluid">
	       <div class="row">
	           <img src="img/footer_service.png" class="img-responsive">
	       </div>
	       <div class="row company">
	           江苏传智播客教育科技股份有限公司 版权所有Copyright 2006-2018, All Rights Reserved 苏ICP备16007882
	       </div>

	   </footer>


	</body>
	</html>
```
