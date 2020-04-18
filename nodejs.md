###  1.what: nodejs

> 1.javascript是运行环境，拥有像浏览器执行JS功能，运行JS的软件。
> 2.事件驱动，非阻塞I/O模型。

### 2.how: nodejs

> 1.地址: https://nodejs.org/en/download/
> 2.install Windows二进制（.zip）
> 3.配置环境变量，NODE_HOME
> 4.cmd node -v

### 3.include: nodejs

> 1.对比浏览器，没有window bom dom
> 2.ECMAScript语法，全局成员，核心模块，第三方模块(donwload npm)

###  4.全局成员

> golbal，timer，require

```javascript
console.log("dirname" + __dirname);
console.log("filename" + __filename);

setTimeout(function () {
    console.log("ok");
}, 1000);

setInterval(function () {
    console.log("okk");
}, 1000);
```

### 5.核心模块

#### fs模块

>异步读写 

```javascript
// 导入核心模块
const fs = require('fs');

// 异步写
const data = 'test write data';
fs.writeFile('./test.txt', data, (err) => {
  if (err) {
  	console.error('write file error');
  	return ;
  }
  console.log('write file success');
});
console.log("writeFile wriet!");

// 异步读
fs.readFile('./test.txt', 'utf8', (err, data) => {
  if (err) {
  	console.error('read file error');
  	return ;
  }
  console.log("data ==> " + data.toString()); 
});
console.log("readFile read!");
```

>同步读写

```javascript
const fs = require('fs');

// 同步写
const write = fs.writeFileSync('./test.txt', 'test write data');
console.log("write ==> " + write); 
console.log("writeFileSync wriet!");

// 同步读
const read = fs.readFileSync('./test.txt','utf-8');
console.log("read ==> " + read.toString()); 
console.log("readFileSync read!");
```

#### http模块

```javascript
// 引入http 模块
const http = require("http");
// 创建http服务器实例
var server = http.createServer(function (req,res) {
    res.writeHead(200,{'content-type':'text/html;charset=utf-8'});
    res.end("你好nodejs");
});
// 监听本地端口 3000
server.listen(3000);
console.log("nodejs server run at 3000");
```

### 6. ES6 新语法

>es6 泛指 ECMAScript 2015 2016 2017
>
>学习地址：http://es6.ruanyifeng.com/

#### let

>let 特点：不会变量提升
>let 特点：不能允许重复定义 同一个作用域
>let 特点：{}内认为是一个局部变量（块级变量）
>let 特点：存在暂时性死区

```javascript
console.log(a);// undefined 变量提升
var a = 10;
var a = 20;// 允许重复定义
if(true){
	var a = 30;// var 只分 全局变量 局部变量
}
console.log(a);//30

var a = 10;
var fn = function () {
    console.log(a);
    // let a 不会变量提升
    // 一个作用域不能重复的定义a
    // 暂时性死区
    let a = 20;
}
fn();
```

#### scope

```javascript
// 产生局部作用域 立即执行 传统
(function(){
	console.log('ok');
})();

// 产生块级作用域 ES6
{
    console.log('ok');
}
```

#### const

>常量：常用的量  且不能修改
>
>不会变量提升，不能重复申明，存在暂时性死区

```javascript
// console.log(PI);
const PI = 3.141592653;
PI = 2;
console.log(PI);

const person = {
    name:'xj',
    age:'35'
};
// person = 10; 修改
console.log(person.age);
// person 是一个 指向 是一个对象 内存存储的地址不会变
person.age = 40;
console.log(person.age);
```

#### string api

````javascript
/*1. api新增*/
let str = 'hello node.js !';

console.log(str.startsWith('he')); //判断字符串是否以he字符串开头
console.log(str.endsWith('!')); //判断字符串是否以!字符串结尾
console.log(str.includes('node')); //判断字符串是否包含 node 字符串

let str1 = 'xxxx ';
console.log(str1.repeat(4)); //生成重复四次拼接的字符串

let str2 = '145'; //卡号  补齐0
console.log(str2.padStart(10, '0')); //字符串前补全后的长度  补全的字符
console.log(str2.padEnd(10, '0')); //字符串后补全后的长度  补全的字符

/*2. 模板字符*/
let person = {
    name:'xj',
    age:35
};
/*<h1>我叫xxx名字，今年xxx岁。</h1>*/
//let personStr = '<h1>我叫'+person.name+'名字，今年'+person.age+'岁。</h1>';
let template = `
    <h1>我叫${person.name}名字</h1>
    <h1>今年${person.age}岁。</h1>
`;
//1. 换行的能力 支持数据替换
console.log(template);
````

#### 解构

>deconstruction
>
>定义变量的时候的结构 和 你指定的数据的结构一致

```javascript
/*1. 数组*/
let arr = ['xj',35,'男'];
let [name,age,gender] = arr;

console.log(name);
console.log(age);
console.log(gender);

/*2. 对象*/
let person = {
    name:'xj',
    age:35,
    gender:'男'
};
let {name:name,age:age,gender:gender} = person;
// 在对象中 如果 属性名称和属性对象的变量 名字一样 省略写 {name} == {name:name}
let {name,gender,age} = person;

console.log(name);
console.log(age);
console.log(gender);
```

#### function/object

```javascript
// 传统定义默认值
let fn = function (x, y) {
	var x = x||0;
	var y = y||0;
	return x + y;
};
console.log(fn());

// ES6定义默认值
let fn = function (x = 0, y = 0) {
	return x + y;
};
console.log(fn());

// 补充：以后尽量使用 表达式方式申明函数，不要使用函数声明式。
function fn() {}
let fn = function () {};

// 属性key与属性value相同，省略一个
// say:function () ==> say()
let name = 'xj';
let person = {
    //name:name,
    name,
    age:35,
    gender:'男',
    // say:function () {
    //     console.log('我是'+this.gender+'生');
    // }
    say(){
        console.log('我叫'+this.name);
    }
};
person.say();
```

#### arrow

>箭头函数
>
>let 函数名 = (传参) => {函数体}
>
>- 箭头函数内部的 this, 永远和 箭头函数外部的 this 保持一致
>- 不可以当构造函数
>- 不可以使用`arguments`对象，如果要用，可以用 rest 参数代替。

```javascript
// 函数体内只有一句话
let fn = function () {
	return 'xj';
};
let fn = () => 'xj';

// 只有一个参数
let fn = (x) => {
	return x;
};
let fn = (x) => x;
let fn = x => x;

// this 指向
let obj = {
    abc: '20',
    say: function () {
        //var _this = this;
        console.log(this.abc);
        //监听函数内的this指向和箭头函数外的this指向一致
        setTimeout(()=>{
            console.log(this.abc);
        });
    }
};
obj.say();
```

#### 参数展开

```javascript
let arr1 = [1,2,3,4];
let arr2 = [5,6,7,8];
let arr = [...arr1,...arr2];
console.log(arr);
```

### 7.模块

>CommonJS: 通用模块规范, 主要由NodeJS具体实现; 根据CommonJS规范, 一个单独的文件就是一个模块。每一个模块都是一个单独的作用域。
>
>CommonJS对模块的定义十分简单：
>
>– 模块引用
>– 模块定义
>– 模块标识

```javascript
// 模块引用示例代码
const math = require(‘math’);

// 模块定义示例代码
// module.exports == exports
// exports.xxx 只能通过属性的方式对外暴露，exports = {}，这个方式修改 exports 所属，是错误的写法。
exports.xxx = function() {};
module.exports = {};

// 模块标识
// 模块标识其实就是模块的名字，也就是传递给require()方法的参数，它必须是符合驼峰命名法的字符串，或者是以.、…开头的相对路径、或者绝对路径。
```

`a.js`

```javascript
let a = 20;
let b = 30;
// add method
let add = function(a, b){
    return a + b;
}
// subtract method
let subtract = function(a, b){
    return a - b;
}

// 方式一：exports
exports.a = a;
exports.add = add;

// 方式二：exports
module.exports = {
    a: a,
    b: b,
    add: add,
    subtract: subtract
}
```

`b.js`

```javascript
const a = require(‘./a.js’); // a.js 与 b.js 在同一目录下
console.log(a.a);
console.log(a.add(12, 34));
```

### 8.npm

1. node package manager: node的包管理工具。

2. 安装NODE 已经自带，不需要单独去安装。

3. javascript package manager  (前端包 node包)管理工具。

4. CommonJS的包规范由包结构和包描述文件两个部分组成。
   包结构: 用于组织包中的各种文件
   包描述文件: 描述包的相关信息，以供外部读取分析

5. 包实际上就是一个压缩文件，解压以后还原为目录。符合规范的目录，应该包含如下文件：

   ```javascript
   – package.json 描述文件 必须
   – bin 可执行二进制文件
   – lib js代码
   – doc 文档
   – test 单元测试
   ```

6. npm命令
>1>.通过npm下载的包都放到node_modules文件夹中，我们通过npm下载的包，直接通过包名引入即可。
>2>.node在使用模块名字来引入模块时，它会首先在当前目录的node_modules中寻找是否含有该模块
>如果有则直接使用，如果没有则去上一级目录的node_modules中寻找
>如果有则直接使用，如果没有则再去上一级目录寻找，直到找到为止
>直到找到磁盘的根目录，如果依然没有，则报错
>
>3>.npm安装设置全局路径：https://blog.csdn.net/weixin_41585557/article/details/79155526


   ```javascript
// – 查看所有版本
npm version
// – 查看版本
npm –v
// - 帮助说明 
npm
// – 搜索模块包
npm search 包名
// – 查看帮助文档 查看某个命令的使用帮助 
npm --help
npm i --help
// –下载当前项目所依赖的包
npm install
// – 在当前目录安装包 包都放到node_modules文件夹中
npm install 包名
// – 全局模式安装包（全局安装的包一般都是一些工具）
npm install 包名 –g
// – 删除一个模块
npm remove 包名
// – 安装包并添加到依赖中 
npm install 包名 --save
// – 从本地安装
npm install 文件路径
// – 从镜像源安装 淘宝镜像
npm install 包名 –registry = 地址
npm install jquery --registry=https://registry.npm.taobao.org
// – 设置镜像源
npm config set registry 地址
// – 安装最新 安装指定版本 安装最新版本
npm i jquery 
npm i jquery@1.12.4 
npm i jquery@latest
// – 卸载指定的包
npm uninstall jquery
npm un jquery
// – 查看单个包远程信息
npm view jquery

// – 配置到淘宝服务器
npm config set registry https://registry.npm.taobao.org
// – 查看 npm 配置信息
npm config list
   ```

7. npm init

```javascript
// – init package.json文件
// – 自定义 package.json 属性
npm init
// – 保持默认的配置
npm init -y

// 文件属性说明
name: (npm-demo) 项目的名称
version: (1.0.0) 0.0.1 项目版本
description: 项目简介
entry point: (index.js) main.js 项目入口
test command: 测试命令，暂且不用关系
git repository: 仓库地址
keywords: 关键字，如果是一个开源项目，则可以填一些关键字被别人在 npm 中搜索到
author: tony 项目的开发者
license: (ISC) 开源协议
```

### 9. web server

获取get，post请求参数示例:

```javascript
const http = require("http");
const url = require('url');
const qs = require('querystring');

const server = http.createServer(function(req, res){
	//res.end('hello nodejs');
	// GET
	let method = req.method;
	if(method == 'GET'){
		 // url.parse(req.url,true) query 是对象
        let {query} = url.parse(req.url, true);
        console.log(query);
	}

	else if(method == 'POST'){
		let postData = '';

		req.on('data', chunk => {
		    postData += chunk.toString()
		  })

		req.on('end', () => {
		 	/*1. 默认的数据格式  键值对字符串  key=value&name=xx */
            /*2. 转成对象*/
            let param = qs.parse(postData);
            console.log(param);
		})
	}
	res.end('end');
});

server.listen(3000, '127.0.0.1');
console.log('web server start http:127.0.0.1:3000');
```

>项目：git clone git@github.com:zhousg/node-feedback-static.git feedback

路由规则

| 地址          | 请求方式 | 业务         |
| ------------- | -------- | ------------ |
| / 或者 /index | get      | 渲染列表页   |
| /publish      | get      | 渲染留言页   |
| /publish      | post     | 处理留言发表 |

```javascript
/*需求*/
/*1. 渲染首页页面*/
/*2. 渲染发表页面*/
/*4. 发表功能*/
/*3. 响应静态资源*/
/*5. 其他响应404*/

const http = require('http');
const fs = require('fs');
const url = require('url');
const path = require('path');
const queryString = require('querystring');
/*模板*/
const artTemplate = require('art-template');
//设置模板引擎的根目录
artTemplate.defaults.root = './views';
//设置模板引擎的后缀名
artTemplate.defaults.extname = '.html';
/*mime*/
const mime = require('mime');

const server = http.createServer();

server.listen(3000, () => {
    console.log('feedback started');
});

//mock data 模拟数据
let feedbackList = [
    {id: 1, nickname: '周杰伦', feedback: '我爱喝奶茶'},
    {id: 2, nickname: '周杰', feedback: '我的鼻孔不是很大'}
];

server.on('request', (req, res) => {
    const {pathname} = url.parse(req.url);
    const method = req.method.toLowerCase();
    /*路由的实现*/
    if (pathname === '/' || pathname === '/index') {
        /*渲染首页*/
        res.end(artTemplate('index', feedbackList));
    } else if (pathname === '/publish') {
        if (method === 'get') {
            /*渲染发布页面*/
            res.end(artTemplate('publish', {}));
        } else if (method === 'post') {
            /*处理发布表单*/
            /*1.接收数据*/
            let data = '';
            req.on('data', (chunk) => {
                data += chunk;
            });
            req.on('end', () => {
                /*data 键值对字符串*/
                /*2.转对象格式*/
                let obj = queryString.parse(data);
                //console.log(obj);
                /*3.非空校验*/
                if (!obj.nickname) {
                    /*重新渲染发布页面 需要提示信息 昵称必须输入*/
                    return res.end(artTemplate('publish', {message: '昵称必须输入'}));
                }
                if (!obj.feedback) {
                    /*重新渲染发布页面 需要提示信息 留言必须输入*/
                    return res.end(artTemplate('publish', {message: '留言必须输入'}));
                }
                /*4.追加到列表数据中*/
                feedbackList.push({
                    id: feedbackList[feedbackList.length - 1].id + 1,
                    nickname:obj.nickname,
                    feedback:obj.feedback
                });
                /*重定向  302 Location /index */
                /*5.去列表页*/
                res.setHeader('Location','/index');
                //res.writeHead(302);
                res.statusCode = 302;
                res.end();
            });
        } else {
            /*404*/
            res.end('404 not found');
        }
    } else if (pathname.startsWith('/public') || pathname.startsWith('/node_modules')) {
        /*请求什么响应什么 处理静态资源*/
        fs.readFile(path.join(__dirname, pathname), (err, data) => {
            if (err) return res.end('404 not found');
            /*响应mime*/
            let type = mime.getType(pathname);
            res.setHeader('Content-Type', type);
            res.end(data);
        });
    } else {
        res.end('404 not found');
    }
});
```

### 10.expressjs 框架

官网：http://www.expressjs.com.cn/

安装：http://www.expressjs.com.cn/starter/installing.html

```javascript
mkdir myapp
cd myapp
npm init -y
npm install express --save
```

get：

```javascript
const express = require('express')
const app = express()
app.listen(3000, () => console.log('Example app listening on port 3000!'))

app.get('/', (req, res) => {
    // 获取get数据
    console.log(req.query);
    res.send('Hello World!'))
}
```

post：

1. 安装 Node.js body parsing middleware.

```javascript
npm install body-parser
```

2. 代码
```javascript
var express = require('express')
var bodyParser = require('body-parser')
 
var app = express()
 
// create application/json parser
var jsonParser = bodyParser.json()
 
// create application/x-www-form-urlencoded parser
var urlencodedParser = bodyParser.urlencoded({ extended: false })
 
// POST /login gets urlencoded bodies
app.post('/login', urlencodedParser, function (req, res) {
  //获取post数据
  console.log(req.body);
  res.send('welcome, ' + req.body.username)
})
 
// POST /api/users gets JSON bodies
app.post('/api/users', jsonParser, function (req, res) {
  // create user in req.body
})
```

静态资源：

```javascript
const express = require('express');
const path = require('path');
const app = express();
app.listen(3000,()=>{ console.log('--------------express start --------------')});
// expres提供的处理静态资源的中间件方法
// 暴露public下的所以文件为静态资源  请求地址不包含public
// 第一个参数 制定的请求路径的前缀
// app.use(express.static('public'))
// http://localhost:3000/static/images/kitten.jpg
app.use('/static',express.static('public'));
app.use('/static',express.static('node_modules'));

app.get('/',(req,res)=>{
    res.sendFile(path.join(__dirname,'./index.html'));
});
```

模板：template

```javascript
const express = require('express');
const path = require('path');
const app = express();
app.listen(3000,()=>{console.log('-------------------------------')});


//启用静态资源处理中间件
app.use('/static',express.static('public'));
app.use('/static',express.static('node_modules'));

const artTemplate = require('express-art-template');
//配置
app.engine('html',artTemplate);
// app.set('view options', {
//     debug: process.env.NODE_ENV !== 'production'
//     //如果电脑上环境变量有一个叫  NODE_ENV 且值 production 证明我是生产环境
//     //  process.env.NODE_ENV !== 'production' false 不开起测试模式 （html压缩）
//     // 配置环境变量后想测试 重启一下生效
// });

//mock 数据
let data = {
    title:'template',
    list:[
        {name:'xj'},
        {name:'xj1'},
        {name:'xj2'},
        {name:'xj3'}
    ]
}

//业务
app.get('/',(req,res)=>{
    //res.sendFile(path.join(__dirname,'./index.html'));
    res.render('index.html',data);
});
```

中间件：类似java中的拦截器

方式一：

```javascript
const express = require('express');
const path = require('path');
const app = express();
app.listen(3000,()=>{console.log('-------------------------------')});

app.all('/index',(req,res,next)=>{
    console.log(req.url);
    // res.duyao = '毒药';  给原有的node的功能 扩展了新的功能
    next();  //下一个环节
});

app.get('/index',(req,res)=>{
    console.log(req.duyao);
    res.send('index get');
});
app.post('/index',(req,res)=>{
    res.send('index post');
});
```

方式二：

```javascript
const express = require('express');
const path = require('path');
const app = express();
app.listen(3000,()=>{console.log('-------------------------------')});

app.use((req,res,next)=>{
    //配置全局的中间配置
    console.log(req.url);
    req.duyao = '毒药';
    next();
});

app.get('/index',(req,res)=>{
    console.log(req.duyao);
    res.send('index get');
});
app.post('/index',(req,res)=>{
    res.send('index post');
});
```

方式三：

```javascript
const express = require('express');
const path = require('path');
const app = express();
app.listen(3000,()=>{console.log('-------------------------------')});

//定义的函数  中间件函数
let middleware = (req,res,next)=>{
    console.log(req.url);
    req.duyao = '毒药';
    next();
};

app.get('/index',middleware,(req,res)=>{
    console.log(req.duyao);
    res.send('index get');
});
app.post('/index',middleware,(req,res)=>{
    res.send('index post');
});
```

### 11. express-CRUD

```javascript
const path = require('path');
/*1.创建服务*/
/*2.使用 body-parser 中间件*/
/*3.使用 express.static 中间件方法*/
/*4.使用 express-art-template 中间件*/
/*5.使用 mysql 模块*/
const express = require('express');
const bodyParser = require('body-parser');
const artTemplate = require('express-art-template');
const mysql = require('mysql');
//1.创建连接
const connection = mysql.createConnection({
    host:'localhost',
    user:'root',
    password:'123456',
    database:'person'
});
//2.建立连接
connection.connect();
//3.操作数据库  具体的业务当中去使用
//4.关闭数据库  不关闭连接  启动服务之后  会有很多数据操作

const app = express();
app.listen(3000,()=>{console.log('--------start-----------')});

//配置中间件 post 数据解析中间件
app.use(bodyParser.urlencoded({extended:false}));
app.use(bodyParser.json());
//配置静态资源处理中间件
app.use('/public',express.static('static'));
app.use('/public',express.static('node_modules'));
//配置渲染模板引擎中间件  默认的模版根目录 views
app.engine('html',artTemplate);
//配置模版根目录
app.set('views',path.join(__dirname,'./pages'));
//引入其他路由模块

/*路由规则*/
/*  / get  index.html 列表*/
app.get('/',(req,res)=>{
    /*1.去数据库获取列表数据*/
    let sql = 'SELECT * FROM user';
    connection.query(sql,(err,data)=>{
        if(err) return res.status(500).send('500 server error');
        /*2. 根据数据去渲染页面*/
        res.render('index.html',{list:data});
    });
});
/*  /save  get   edit.html  添加页 */
app.get('/save',(req,res)=>{
    res.render('edit.html',{user:{}});
});
/*  /save  post  添加功能  跳转列表页 */
app.post('/save',(req,res)=>{
    //获取post请求数据
    let body = req.body;
    /*1. 去数据库执行添加操作*/
    let sql = `INSERT INTO user(username,gender,hobby) 
    VALUES("${body.username}",${body.gender},"${body.hobby}")`;
    connection.query(sql,(err,data)=>{
        console.log(err);
        //保存失败
        if(err) return res.status(500).send('500 server error');
        //保存成功
        res.redirect('/');
    });
});
/*  /edit  get    edit.html  修改页 */
app.get('/edit',(req,res)=>{
    connection.query('SELECT * FROM user WHERE id='+req.query.id,(err,data)=>{
        if(err) return res.status(500).send('500 server error');
        //data 对象{} 一个数组[{}]
        res.render('edit.html',{user:data[0]});
    });
});
/*  /edit  post  修改功能  跳转列表页 */
app.post('/edit',(req,res)=>{
    let body = req.body;
    let sql = 'UPDATE user SET username=?,gender=?,hobby=? WHERE id = ?';
    connection.query(sql,[body.username,body.gender,body.hobby,body.id],(err,data)=>{
        if(err) return res.status(500).send('500 server error');
        res.redirect('/');
    });
});
/*  /del   get   删除功能  跳转列表页 */
app.get('/del',(req,res)=>{
    connection.query('DELETE FROM user WHERE id='+req.query.id,(err,data)=>{
        if(err) return res.status(500).send('500 server error');
        res.redirect('/');
    });
});
```

