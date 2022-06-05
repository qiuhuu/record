## 命名规范

### 普通变量命名规范

- 命名方法 ：驼峰命名法
- 命名规范 ：

> 1. 命名必须是跟需求的内容相关的词，如

```javascript
let  productPageDetail = "产品详情页面";
```

> 1. 命名是复数的时候需要加s,如

```javascript
const productList = new Array(); 
```

### 常量

- 命名方法 : 全部大写
- 命名规范 : 使用大写字母匈牙利式命名法。



```javascript
const MAX_COUNT = 10
const URL = 'https://www.cupshe.com/'
```

### 组件命名规范（驼峰式命名）

- 公用组件以cupshe_AR(公司+项目名)开头如：



```javascript
cupsheAR-TopBar,
cupsheDE-TopBar
```

- 页面内部组件以组件模块名简写为开头，Item 为结尾，如：



```javascript
addToCartItem,
checkoutItem
```

### method 方法命名命名规范

- 匈牙利式命名，统一使用动词或者动词+名词形式



```javascript
get_user_list,
submit_cart_product
```

- 请求数据方法，以 data 结尾



```javascript
get_product_list_data
get_user_data
```

### views下的文件命名

- 尽量是名词,且使用驼峰命名法



```javascript
productDetailPage
```

### props 命名

- 在声明 prop 的时候，使用驼峰命名法，在模板中使用 kebab-case



```html
<script>
props: {
    greetingText: String
}
</script>
<welcome-message greeting-text="hi"></welcome-message>
```



## 常见目录规范

```javascript
src                               	 源码目录
|-- api                              接口，统一管理
|-- assets                           静态资源，统一管理
|-- components                       公用组件，全局文件
|-- filters                          过滤器，全局工具
|-- icons                            图标，全局资源
|-- datas                            模拟数据，临时存放
|-- lib                              外部引用的插件存放及修改文件
|-- mock                             模拟接口，临时存放
|-- router                           路由，统一管理
|-- store                            vuex, 统一管理
|-- views                         	 视图目录
|   |-- staffWorkbench               视图模块名
|   |-- |-- staffWorkbench.vue       模块入口页面
|   |-- |-- indexComponents          模块页面级组件文件夹
|   |-- |-- components               模块通用组件文件夹
```



## 注释规范

###### 务必添加注释列表

> 1. 公共组件使用说明
> 2. 各组件中重要函数或者类说明
> 3. 复杂的业务逻辑处理说明
> 4. 特殊情况的代码处理说明,对于代码中特殊用途的变量、存在临界值、函数中使用的 hack、使用
> 5. 了某种算法或思路等需要进行注释描述
> 6. 多重 if 判断语句
> 7. 注释块必须以/**（至少两个星号）开头**/
> 8. 单行注释使用//