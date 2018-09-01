## 用户积分扣除接口 {#用户积分扣除接口}

#### 输入参数（Get请求方式传参）

|  | 是否必须 | 参数类型 | 限制长度 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |
| uid | yes | string | 255 | 用户唯一性标识，唯一且不可变 |
| credits | yes | long | 20 | 本次兑换扣除的积分 |
| itemCode | no | string | 255 | 自有商品商品编码\(非必须字段\) |
| appKey | yes | string | 255 | 接口appKey，应用的唯一标识 |
| timestamp | yes | string | 20 | 1970-01-01开始的时间戳，毫秒为单位。 |
| description | yes | string | 255 | 本次积分消耗的描述\(带中文，请用utf-8进行url解码\) |
| orderNum | yes | string | 255 | 兑吧订单号\(请记录到数据库中\) |
| type | yes | string | 255 | 兑换类型：alipay\(支付宝\), qb\(Q币\), coupon\(优惠券\), object\(实物\), phonebill\(话费\), phoneflow\(流量\), virtual\(虚拟商品\),game\(游戏\), hdtool\(活动抽奖\),sign\(签到\)所有类型不区分大小写 |
| facePrice | no | int | 11 | 兑换商品的市场价值，单位是分，请自行转换单位 |
| actualPrice | yes | int | 11 | 此次兑换实际扣除开发者账户费用，单位为分 |
| ip | no | string | 255 | 用户ip，不保证获取到 |
| waitAudit | no | boolean |  | 是否需要审核\(如需在自身系统进行审核处理，请记录下此信息\) |
| params | no | string | 255 | 详情参数，不同的类型，返回不同的内容，中间用英文冒号分隔。\(支付宝类型带中文，请用utf-8进行解码\) 实物商品：返回收货信息\(姓名:手机号:省份:城市:区域:详细地址\)、支付宝：返回账号信息\(支付宝账号:实名\)、话费：返回手机号、QB：返回QQ号 |
| sign | yes | string | 255 | MD5签名，详见[签名规则](http://docs.duiba.com.cn/tech_doc_book/appendix/sign_rule.html) |

#### 响应参数 {#响应参数}

| 参数 | 是否必须 | 参数类型 | 限制长度 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |
| status | yes | string | 255 | 扣积分结果状态，回复ok或者fail （不要使用0和1） |
| errorMessage | no | string | 255 | 出错原因 |
| bizId | yes | string | 255 | 开发者的订单号\(唯一且不重复，如果失败情况，该值可以不传\) |
| credits | yes | long | 20 | 用户积分余额 |

请按JSON格式返回结果。

#### 响应示例 {#响应示例}

```
成功：
{
    "status":"ok",
    "errorMessage":" ",
    'bizId':"20140730192133033",
    "credits":"100"
}
```

```
失败：
{
    "status":"fail",
    "errorMessage":"失败原因（显示给用户）",
    "credits":"0"
}
```

#### 代码示例 {#代码示例}

java

[**点击下载（java开发包）**](https://github.com/duiba-Tech/duiba-java-sdk/archive/master.zip)

```
@RequestMapping
(
"/consume"
)

@ResponseBody
public
 String 
consume
(HttpServletRequest request)
{
     CreditTool tool = 
new
 CreditTool(appKey, appSecret);
     
boolean
 success = 
false
;
     String errorMessage = 
""
;
     String bizId =
null
;
     Long credits=
0L
;
 
try
 {
     CreditConsumeParams params = tool.parseCreditConsume(request);

     bizId = todo(); 
//开发者业务订单号，保证唯一不重复

     credits = getCredits(); 
// getCredits()是根据开发者自身业务，获取的用户最新剩余积分数。

     success = 
true
;
 } 
catch
 (Exception e) {
     success = 
false
;
     errorMessage = e.getMessage();
     e.printStackTrace();
 }
     CreditConsumeResult ccr = 
new
 CreditConsumeResult(success);
     ccr.setBizId(bizId);
     ccr.setErrorMessage(errorMessage);
     ccr.setCredits(credits);
     
return
 ccr.toString();
//返回扣积分结果json信息

 }

```

php

**开发工具包 ：**[**点击打开**](http://home.duiba.com.cn/php.html?_blank)

```
/*
*  积分消耗请求的解析方法
*  当用户进行兑换时，兑吧会发起积分扣除请求，开发者收到请求后，可以通过此方法进行签名验证与解析，然后返回相应的格式
*/
function
parseCreditConsume
($appKey,$appSecret,$request_array)
{
    
if
($request_array[
"appKey"
] != $appKey){
        
throw
new
Exception
(
"appKey not match"
);
    }
    
if
($request_array[
"timestamp"
] == 
null
 ){
        
throw
new
Exception
(
"timestamp can't be null"
);
    }
    $verify=signVerify($appSecret,$request_array);
    
if
(!$verify){
        
throw
new
Exception
(
"sign verify fail"
);
    }
    $ret=
array
(
"appKey"
=
>
$request_array[
"appKey"
],
"credits"
=
>
$request_array[
"credits"
],
"timestamp"
=
>
$request_array[
"timestamp"
],
        
"description"
=
>
$request_array[
"description"
],
"orderNum"
=
>
$request_array[
"orderNum"
]);
    
return
 $ret;
}

```

.net

**开发工具包：**[**点击打开**](http://home.duiba.com.cn/net.html?blank)

```
/*
*  积分消耗请求的解析方法,重点在于方法中的三重验证
*  当用户进行兑换时，兑吧会发起积分扣除请求，开发者收到请求后可以通过此方法进行签名验证与解析，然后返回相应的格式
*/
public
 Hashtable 
parseCreditConsume
(
string
 appKey,
string
 appSecret,HttpRequest request
)

{
    
if
(!request.Params[
"appKey"
].Equals(appKey)){
        
throw
new
 Exception(
"appKey not match"
);
    }
    
if
(request.Params[
"timestamp"
] == 
null
 ){
        
throw
new
 Exception(
"timestamp can't be null"
);
    }
    Hashtable hshTable = duiba.GetUrlParams(request);

    
bool
 verify=duiba.SignVerify(appSecret,hshTable);
    
if
(!verify){
        
throw
new
 Exception(
"sign verify fail"
);
    }
    
return
 hshTable;
}

```



