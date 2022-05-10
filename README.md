

后台登录用户名密码：

用户名username：admin 
密码：100809

### 该项目基于[海风小店](https://github.com/iamdarcy/hioshop-admin)修改
### 海风小店，开源商城（后台管理端VUE）

+ 基于开源项目NideShop重建，精简了一些功能的同时完善了一些功能，并重新设计了UI
+ 测试数据来自上述开源项目
+ 服务端api基于Ｎode.js+Express.js+MySQL
+ 后台管理 基于VUE.js+element-ui


#### 本项目需要配合  
服务端： https://github.com/sshaunn/season-ecommerce-server-express
微信小程序：https://github.com/sshaunn/season-ecommerce-miniprogram

线上demo：http://hiolabs.com/demo/#/login

用户名：hiolabs  
密码：hiolabs

### 项目截图
+ Dashboard

<img width="900" src="http://git.hiolabs.com/github/dashboard.jpg"/>

+ 订单

<img width="900" src="http://git.hiolabs.com/github/order.jpg"/>

+ 电子面单生成

<img width="900" src="http://git.hiolabs.com/github/express2.jpg"/>

+ 商品管理

<img width="900" src="http://git.hiolabs.com/github/goods.jpg"/>

+ 购物车

<img width="900" src="http://git.hiolabs.com/github/cart.jpg"/>

+ 用户

<img width="900" src="http://git.hiolabs.com/github/user.jpg"/>

+ 运费模板

<img width="900" src="http://git.hiolabs.com/github/template.jpg"/>

+ 运费模板详情页

<img width="900" src="http://git.hiolabs.com/github/template2.jpg"/>

### 本地开发环境配置
```
+ 安装依赖
```
npm install

```
+ 安装依赖后启动后会出现的第一个问题：
<img width="600" src="http://git.hiolabs.com/github/error.jpg"/>

这个问题是Element-ui自带的。解决方法：

在node_modules 搜索:  div class="el-form-item__label-wrap" style={style}  
然后在语句中加上单引号就可以了。

<img width="600" src="http://git.hiolabs.com/github/before.jpg"/>

<img width="600" src="http://git.hiolabs.com/github/after.jpg"/>

<br></br>
+ 安装依赖后启动后会出现的第二个问题：
````
Uncaught SyntaxError: Unexpected token export
````
这个问题是由于 ````vue-printjs````模块输出方式与````Electron Js````不兼容导致。解决方法如下：

在 ````node_modules/vue-printjs/print.js```` 文件最中搜索 ````export default MyPlugin````（最后一行）
将其修改为````module.exports = MyPlugin```` 即可

+ 启动
```
npm run dev
```
+ build 打包成静态文件
```
npm run build:web 或者 sudo npm run build:web

```

生成的静态文件在dist的web文件夹中，上传到服务器就可以在浏览器中打开了。

### 功能列表
+ 订单管理：查看，修改订单价格，发货，生成电子面单，修改订单状态
+ 商品管理：添加、修改、删除商品，添加商品分类
+ 购物车：可以看到用户加入购物车的情况
+ 用户列表：用户的一些使用踪迹
+ 店铺设置：广告列表，公告管理，运费模板（可以根据地区设置相应的运费），管理员



