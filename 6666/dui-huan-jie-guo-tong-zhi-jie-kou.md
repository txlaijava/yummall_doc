## 订单兑换成功/失败消息的接收接口 {#订单兑换成功失败消息的接收接口}

####  请求参数（Get请求方式传参）

| 参数 | 是否必须 | 参数类型 | 限制长度 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |
| appKey | yes | string | 255 | 接口appKey，应用的唯一标识码 |
| timestamp | yes | string | 20 | 1970-01-01开始的时间戳，毫秒。 |
| uid | yes | string | 255 | 用户唯一标识，唯一且不可变 |
| success | yes | boolean |  | 兑换是否成功，状态是true和false |
| errorMessage | no | string | 255 | 出错原因\(带中文，请用utf-8进行解码\) |
| orderNum | yes | string | 255 | 兑吧订单号 |
| bizId | no | string | 255 | 开发者的订单号 |
| sign | yes | string | 255 | 签名 |

#### 响应参数： {#响应参数：}

开发者服务器端收到通知并处理完成后，请返回纯文本的`ok`字符串，两边不带空格，忽略大小写，云猫验证到响应为 ok 后会标记通知成功。

如果响应为非 ok 字符串，兑吧会在**24小时内**最多重试**8**次通知。  
通知时间间隔为：2m、10m、10m、1h、2h、6h、15h。

注：出于网络异常的可能性，云猫平台可能会对开发者进行重复通知，开发者务必确保一笔订单不进行重复处理，**否则将产生严重bug**，详细原因参考下述：`重复通知处理`

#### 重复通知处理 {#重复通知处理}

> 由于网络具有不稳定的特性，当云猫平台向开发者服务器发送成功/失败通知时，有可能存在云猫平台发送了通知，开发者收到了通知并进行了处理。
>
> 若此时出现网络故障，开发者响应云猫平台失败，云猫平台没有收到开发者的响应，云猫平台会认为开发者没有收到通知，于是进行重复通知。
>
> 此时开发者收到通知后，务必先确认此订单是否已经处理过。如果已经处理过，则忽略此通知，并响应ok。 如果此时开发者忽略订单是否已经被处理过，而直接进行处理，将导致开发者反复向用户返还积分，导**致损失**！

#### 请勿限制通知等待时长 {#请勿限制通知等待时长}

> 有部分订单：如实物类订单，需等待发货之后才会确认订单成功，此类订单可能会隔比较长时间才发订单结果通知。如果限制了通知等待时长会导致部分订单接收不到结果。

#### 代码示例 {#代码示例}

java

```java

CreditTool tool=new CreditTool(appKey, appSecret);

try {
    CreditNotifyParams params= tool.parseCreditNotify(request);//利用tool来解析这个请求
    String orderNum=params.getOrderNum();
    if(params.isSuccess()){
        //兑换成功
    }else{
        //兑换失败，根据orderNum，对用户的金币进行返还，回滚操作
    }
} catch (Exception e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
}
```

php

```php
function parseCreditNotify($appKey,$appSecret,$request_array){
    if($request_array["appKey"] != $appKey){
        throw new Exception("appKey not match");
    }
    if($request_array["timestamp"] == null ){
        throw new Exception("timestamp can't be null");
    }
    $verify=signVerify($appSecret,$request_array);
    if(!$verify){
        throw new Exception("sign verify fail");
    }
    $ret=array("success"=>$request_array["success"],"errorMessage"=>$request_array["errorMessage"],
        "uid"=>$request_array["uid"],"bizId"=>$request_array["bizId"]);
    return $ret;
}
```

.net

```java
function parseCreditNotify(string appKey,string appSecret,HttpRequest request)
{
    if(!request.Params["appKey"].Equals(appKey)){
        throw new Exception("appKey not match");
    }
    if(request.Params["timestamp"] == null ){
        throw new Exception("timestamp can't be null");
    }
    Hashtable hshTable = duiba.GetUrlParams(request);

    bool verify=duiba.SignVerify(appSecret,hshTable);
    if(!verify){
        throw new Exception("sign verify fail");
    }
    return hshTable;
}
```

  


  


