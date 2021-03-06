# 免登录URL生成

#### 输入参数\(Get请求方式传参\) {#输入参数get请求方式传参}

| 参数 | 是否必须 | 参数类型 | 限制长度 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |
| uid | yes | string | 255 | 用户唯一性标识，对应唯一一个用户且不可变 （用not\_login作为uid标记游客用户） |
| credits | yes | long | 20 | 用户积分余额 |
| appKey | yes | string | 255 | 接口appKey，应用的唯一标识 |
| timestamp | yes | String | 20 | 1970-01-01开始的时间戳，毫秒为单位。 |
| **redirect** | **no** | string | 255 | 登录成功后的重定向地址，可以直达积分商城内的任意页面,如果不带redirect参数，默认跳转到积分商城首页 |
| sign | yes | string | 255 | MD5签名 |

### 注意事项 {#注意事项}

1. 为了确保客户端每次请求到都是最新的免登录url，客户端每次向服务器发的请求不能是固定的，以避免请求被缓存。

2. 免登录url经过签名，**该url地址5分钟失效，请务必在生成地址后立即使用**，使用后页面会重定向进入积分商城，登录状态24小时有效。

3. 开发者服务器端需要开发一个支持重定向的接口实现动态生成免登录url地址，该接口地址配置在客户端，用户通过点击该地址访问积分商城。

### 代码示例 {#代码示例}

java

```java
CreditTool tool=new CreditTool(appKey, appSecret);
Map params=new HashMap();
params.put("uid","userId001");
params.put("credits","100");
if(redirect!=null){
 // redirect是目标页面地址，如果要跳转到积分商城指定页面，redirect地址就是目标页面地址
 // 此处请设置成一个外部传进来的参数，方便运营灵活配置
 params.put("redirect", redirect);
}
String url=tool.buildUrlWithSign("https://www.yummall.cn/autoLogin/autologin?",params);
//此url即为免登录url
```

php

```php
$url=buildRedirectAutoLoginRequest($appKey,$appSecret,$uid,$credits,$redirect)
```

.net

```java
string url = "https://www.yummall.cn/autoLogin/autologin";
string uid = "userId001";
int credits = 9999;

Hashtable hshTable = new Hashtable();
hshTable.Add("uid", uid);
hshTable.Add("credits", credits);
if(redirect!=null){
 //redirect为商城直达的目标页面地址，如果要跳转的目标页面不是积分商城首页，改地址必须配置
 hshTable.Add("redirect", redirect);
}
url = duiba.BuildUrlWithSign(url, hshTable, appKey, appSecret);
```



