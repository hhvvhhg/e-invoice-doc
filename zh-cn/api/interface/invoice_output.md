## 前提
所有的请求和响应 必须都使用 utf-8 字符编码

喔噻平台的功能图
![](../../img/wosai_invoice_platform_arch.png?raw=true)


喔噻开票平台, 提供3种接口,分别为
1. 开票接口
2. 开票状态被动查询接口

以及 1 种, 主动推送的 Callback (webhook)
3. 开票结果主动通知 Callback


## 接入说明
 > * 第1步: 商户准备卖家纳税人识别号、开票卖家抬头、税率、商户简称、商户门店列表(商户门店唯一标识、商户简称、门店开票卖家抬头)；
 > * 第2步: 根据[对接前准备](https://doc.shouqianba.com/invoice/zh-cn/bussiness.html)开通自己的 开票应用,并获得 app_id
 > * 第3步: 提交商户资料，获取[商户平台](http://s.shouqianba.com)登录账号和密码,在[对接前准备](https://doc.shouqianba.com/invoice/zh-cn/bussiness.html)
 通过
 [激活接口](https://doc.shouqianba.com/invoice/zh-cn/api/interface/activate.html)
 [签到接口](https://doc.shouqianba.com/invoice/zh-cn/api/interface/checkin.html)

  ```
  进行获取 terminal_sn 和 terminal_key
  ```

 > * 第4步:

 ```
 如果是接口开票, 直接阅读 开票接口, 不需要阅读 二维码生成规范

 如果需要通过生成二维码进行开票
 应在开票前先基于 <发票二维码生成规范> 生成存有开票预确认 http 网页地址的<二维码>, 然后将二维码打印在收银票据上；
 用户可以使用移动终端, 访问扫描这个二维码, 访问商户应用网页, 进行开票前确认或补完信息, 然后再通过页面的按钮，发起开票请求，<商户应用后端>收到<商户应用前端(H5)>的开票请求后，整合必要的其他必要参数，再由应用后端向<喔噻电子发票平台>发起开票请求
 (二维码中的 "开票预确认" 的网页, 由商户应用层自行开发, 收钱吧仅提供 二维码生成规范)
 ```

 > * 第5步: 开发接收通知的接口，订阅收钱吧推送的开票结果信息。


## 开票

开票的时序图

![](../../img/invoice_normal.png?raw=true)

![](../../img/invoice_error_user_return_fail.png?raw=true)

![](../../img/invoice_error_call_user_no_response.png?raw=true)



### 开票接口
 - 接口地址：{api_domain}/api/invoice/apply/v2
 - 访问方式：post
 - 签名校验：参照2.1.4
 - 参数说明：

<div style="display:none;">
<!---sum_price|合计金额，不含税金额(新版中为必传) 当开红票时，该字段为负数,单位为分|string|Y|100000)-->
<!---sum_tax|合计税额 当开红票时，该字段为负数,单位为分|string|Y|17000-->
</div>

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
user_from|发起开票请求操作的用户来源|string|Y|0: 未知; 1: 支付宝; 2: 微信
user_uid|对应平台的 user-openId(用户唯一标识)|string|Y|当 user_from 识别为 "支付宝"、"微信"时, 这个值是 对应平台的 user-openId(用户唯一标识), 否则应设置为 终端号(terminal_sn)
terminal_sn|终端号|string|Y|
recommandation_info|推荐信息 由商家给出, 用于开票成功后给客户推送邮件时的特殊附加 url; 如果展示url, 使用格式如: <br/>``` [关注我们的官网](http://xxxx.xxx.xxx) ``` <br/>如果是要直接展示二维码, 使用格式如: <br/>``` [#qrcode](http://xxxx/xxx/xxx) ```|string(150)|N|
notify_url|开票请求回调地址|string|Y|
client_sn|商户系统原始订单号|string|Y|必须在商户系统内唯一；且长度不超过32字节
client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节
client_time|商户系统订单完成时间|int|Y|timestamp,单位毫秒
business_type|开票对象业务类型|string(1)|N|默认：0。对于商家对个人开具，为0;对于商家对企业开具，为1;
invoice_type|开票类型|string(10)|Y|0:BLUE-蓝票(开蓝票), 1:RED-红票(红冲)
apply_from|申请发起方角色类型|string|N|0: PAYEE 和 1: PAYER, 默认值是 1: "PAYER"
payer_register_no|付款方税务纳税人识别号。对企业开具电子发票时必填。目前北京地区暂未开放对企业开具电子发票，若北京地区放开后，对于向企业开具的情况，付款方税务登记证号和名称也不能为空,注：根据国家税务总局公告2017年第16号公告，2017年7月1日起，增值税普通发票必须填写纳税人识别号，否则无法作为企业内部报销凭证。|string(20)|N|2015020123123
invoice_amount|开票金额； 当开红票时，该字段为负数, 单位为分 |string|Y|117000
invoice_memo|发票备注，有些省市会把此信息打印到PDF中,会显示在票张的右下角备注栏|string(200)|N|电子发票测试
invoice_time|开票日期, 格式"YYYY-MM-DD HH:SS:MM"|Date|Y|2015-05-21 12:00:00
normal_invoice_code|原发票代码(开红票时传入)|string(12)|N|111100000000
normal_invoice_no|原发票号码(开红票时传入)|string(8)|N|00004349
payer_address|购买方地址，若填写则显示在票张左上的购买方地址栏|string(100)|N|浙江省杭州市余杭区文一西路xxx号
payer_bankaccount|购买方开票开户银行账号，若填写则显示在票张左上的开户行及账号栏|string(100)|N|123412341234
payer_bank_name|付款方开票开户银行|String(100)|N|上海银行
payer_email|消费者电子邮箱，用于接受电子发票票张归集邮件|string|Y|mytest@xxx.com
payer_name|购买方名称, 对应发票抬头，显示在票张左上的购买方名称栏|string(100)|Y|付款方名称, 对应发票抬头
sum_tax|合计税额 当开红票时，该字段为负数,单位为分|string|Y|17000
sum_price|合计金额，不含税金额(新版中为必传) 当开红票时，该字段为负数,单位为分|string|Y|100000)
user_phone|消费者联系电话|string(16)|Y|18234561212
payer_phone|购买方联系电话，若填写则显示在票张左上的购买方电话栏|string(16)|N|18234561212 或者 0512-55553333
invoice_items|开票商品明细信息|[]|Y|参考开票明细信息
reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }
payway|支付通道唯一标识|string(20)|N|1:支付宝 3:微信 4:百度钱包 5:京东钱包 6:qq钱包 100:现金 101:银联卡 110:银行卡; 如果经由多种支付完成，则可用英文 "," 进行连接， 比如： 1,100
payer_uid|开票用户 唯一标识(id), 如：支付宝id,微信open_id等|string(64)|N|开票用户唯一标识


   - 开票商品明细信息(invoice_items 中的每个 item)

<div style="display:none;">
<!---price|商品不含税单价，单位为分,格式：10000。新版电子发票，折扣行此参数不能传，非折扣行必传；红票、蓝票都为正数|string|N|10000-->
<!---sum_price|总价, 不含税金额，格式：100.00； 当开红票时，该字段为负数|string|Y|100000)-->
<!---tax|税额； 当开红票时，该字段为负数, 单位是分|string|Y|17000-->
</div>

名称|含义|类型|必填|备注
----|:---|:---|:--:|--------
item_no|商品税收编码|string|N|商品税收编码最长度必须为19位或21位数字，折扣行与被折扣行需保持一致
item_name|发票项目名称，即商品名称，将显示在票张货物或应税劳务、服务名称栏|string|Y|电视机，折扣行与被折扣行需保持一致
quantity|数量, 默认为1。新版电子发票，折扣行此参数不能传，非折扣行必传； 当开红票时，该字段需为负数|string|N|10，折扣行与被折扣行需保持一致，折扣行不填
row_type|发票行性质。0表示正常行，1表示折扣行，2表示被折扣行。比如充电器单价100元，折扣10元，则明细为2行，充电器行性质为2，折扣行性质为1。如果充电器没有折扣，则值应为0|string|Y|0
specification|规格型号,若填写将显示在票张规格型号栏|string|N|X100
tax_rate|税率。税率只能为0或0.03或0.04或0.06或0.11或0.13或0.17|string|Y|0.17
price|商品不含税单价，单位为分,格式：10000。新版电子发票，折扣行此参数不能传，非折扣行必传；红票、蓝票都为正数|string|N|10000，折扣行不填
sum_price|总价, 不含税金额，格式：100.00； 当开红票时，该字段为负数|string|Y|100000
tax|税额； 当开红票时，该字段为负数, 单位是分|string|Y|17000，折扣行税额=不含税金额*税率
unit|单位，将显示在票张单位栏；。新版电子发票，折扣行不能传，非折扣行必传|string|N|台，折扣行不填
amount|价税合计。(等于sumPrice和tax之和) 当开红票时，该字段为负数;单位是分|string|Y|117000，折扣行商品价税合计金额=不含税金额+税额
zero_rate_flag|0税率标识，只有税率为0的情况才有值，0=出口零税率，1=免税，2=不征收，3=普通零税率|string|N|1，折扣行不填

- 折扣行相关说明：

  > 1.折扣行只能对应一行被折扣行，且折扣行必须在被折扣行下一行
  > 2.折扣行与被折扣行“货物劳务名称”、“商品税收编码”、“商品规格型号”、“税率”需保持一致
  > 3.冲红时，只需要将正常行和对应的折扣行金额相加取负值即可



 - 蓝票参数示例：

```json
{
	"client_sn": "testhyb",
	"client_task_sn": "testhyb001",
	"client_time": "1526011200000",
	"invoice_amount": "1044",
	"invoice_time": "2018-05-11 12:00:00",
	"invoice_type": "0",
	"notify_url": "http://uinvoice-alipay-morkserver.test.shouqianba.com/v2/push",
	"payer_email": "1369617918@qq.com",
	"payer_name": "feixiang",
	"payer_phone": "18014891021",
	"payer_register_no": "91500000747150346A",
	"sum_price": "900",
	"sum_tax": "144",
	"terminal_sn": "2100216260002212407",
	"apply_from": "0",
	"user_from": "2",
	"user_uid": "5e0bce11894311e7bcb0d0a637ea2ec3",
	"user_phone": "12323244323",
	"invoice_items": [{
		"amount": "1160",
		"item_name": "礼品卡",
		"item_no": "1040201080000000000",
		"price": "100",
		"quantity": "10",
		"row_type": "2",
		"specification": "Z",
		"sum_price": "1000",
		"tax": "160",
		"tax_rate": "0.16",
		"unit": "件"
	}, {
		"amount": "116",
		"item_name": "礼品卡",
		"item_no": "1040201080000000000",
		"row_type": "1",
		"sum_price": "100",
		"specification": "Z",
		"tax": "16",
		"tax_rate": "0.16"
	}]
}
```

- 红冲参数示例
```json
{
	"client_sn": "testhyb",
	"client_task_sn": "testhyb056",
	"client_time": "1526011200000",
	"invoice_amount": "-1044",
	"invoice_items": [
		{
			"amount": "-1160",
			"item_name": "礼品卡",
			"item_no": "1040201080000000000",
			"price": "100",
			"quantity": "-10",
			"row_type": "2",
			"specification": "Z",
			"sum_price": "-1000",
			"tax": "-160",
			"tax_rate": "0.16",
			"unit": "件"
		}, {
			"amount": "-116",
			"item_name": "礼品卡",
			"item_no": "1040201080000000000",
			"row_type": "1",
			"sum_price": "-100",
			"tax": "-16",
			"tax_rate": "0.16"
		}
	],
	"invoice_time": "2018-05-11 12:00:00",
	"invoice_type": "1",
	"notify_url": "http://uinvoice-alipay-morkserver.test.shouqianba.com/v2/push",
	"payer_email": "1369617918@qq.com",
	"payer_name": "feixiang",
	"payer_phone": "18014891021",
	"payer_register_no": "91500000747150346A",
	"normal_invoice_code": "150003533340",
	"normal_invoice_no": "42018793",
	"sum_price": "-900",
	"sum_tax": "-144",
	"terminal_sn": "2100216260002212407",
	"apply_from": "0",
	"user_from": "2",
	"user_uid": "5e0bce11894311e7bcb0d0a637ea2ec3",
	"user_phone": "12323244323"
}
```

 - 返回说明：

|字段名|字段含义|取值|必要|备注|
|----|:---|:---|:--:|--------|
|task_status|开票状态| |N| |
|client_sn|商户系统订单号|string|Y|必须在商户系统内唯一；且长度不超过32字节|
|client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节|
|task_sn|开票任务唯一标识| |N|开票申请成功的时返回|
|status|流水状态|见《业务结果码》|N| |
|reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }|

 - 正常返回示例

```json
{
    "biz_response": {
        "data": {
            "client_sn": "22000000012",
            "client_task_sn": "22000009989",
            "status": "SUCCESS",
            "task_sn": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "task_status": "CREATED"
        },
        "result_code": "INVOICE_SUCCESS"
    },
    "result_code": "200"
}

```

### 开票结果查询接口
 - 接口地址：{api_domain}/api/invoice/query/v2
 - 访问方式：post
 - 签名校验：参照2.1.4
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|terminal_sn|终端号|string|Y| |
|client_sn|商户系统订单号|string|N|必须在商户系统内唯一；且长度不超过32字节, client_sn和task_sn任意传一个|
|client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节|
|task_sn|开票任务唯一标识|string|N|开票申请成功的时返回|

 - 参数示例

```json
{
    "terminal_sn": "2200000001",
    "client_sn": "22000000012",
    "client_task_sn": "22000009989",
    "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

}
```

 - 返回结果 进行中

```json
{
    "result_code": "200",
    "biz_response": {
        "result_code": "INVOICE_IN_PROGRESS",
        "data": {
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "client_sn": "22000000012",
            "client_task_sn": "22000009989",
            "terminal_sn":"2200000001"
        }
    }
}
```

 - 返回结果 成功

```json
{
    "result_code": "200",
    "biz_response": {
        "result_code": "SUCCESS",
        "data": {
            "terminal_sn":"2200000001",
            "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
            "client_sn": "22000000012",
            "client_task_sn": "22000009989",
            "finish_time": "1492506702864",
            "invoice_date": "2017-01-12",
            "invoice_code": "150003528888",
            "invoice_no": "50877603",
            "anti_fake_code": "CF6B2F6168420008",
            "invoice_amount": "117000",
            "file_path":"demo",
            "payer_name": "发票抬头",
            "payer_mobile": "0512-55553333",
            "payer_phone": "0512-55553333",
            "user_phone:": "18268888888"
            "payer_email":"123@qq.com",
            "payer_register_no": "9133010060913454XP",
            "invoice_type":"0",
            "reflect":reflect_struct
        }
    }
}
```



### 开票结果主动通知, 回调(web hook)规范, (被回调的接口由商户实现并提供)
这里的地址 , 已在 开票接口的 notify_url 中传入

    商户提供接口接收 -> 收钱吧 开票成功（发票信息）或开票失败的通知信息。
    开票完成后，收钱吧调用商户的接口推送开票结果信息，推送 1 次 商户返回带有 SUCCESS 的响应参数视为成功

 - 回调接口地址：{user_callback_api_domain}/v2
 - 回调请求类型：post
 - 回调请求参数说明：


|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|client_sn|商户系统订单号|string|Y| |
|client_task_sn|商户系统开票任务流水号|string|Y|必须在商户系统内唯一; 且长度不超过32字节|
|task_sn|开票任务号|string|Y|开票的任务号|
|finish_time|任务完成时间|string|Y|"1492506702864"|
|channel_finish_time|通道完成时间|string|N|"1492506305637"|
|invoice_date|开票日期|string|Y|2017-01-12|
|invoice_code|发票代码|string|N|开票成功必传, 1231231234|
|invoice_no|发票编号|string|N|开票成功必传, 123123|
|invoice_amount|开票金额； 当开红票时，该字段为负数, 单位为分 |string|Y|117000|
|invoice_type|开票类型|string(10)|Y|0:BLUE-蓝票(开蓝票), 1:RED-红票(红冲)|
|device_no|税控设备编号(新版电子发票有)|string|N|sw1231|
|file_path|发票PDF的下载地址(仅在单个查询接口上显示，批量查询不显示)|string|N|demo|
|file_date_type|文件类型(pdf,jpg,png)|string|Y|jpg|
|ciphertext|发票密文，密码区的字符串|string|N|demosdffsd-32432|
|anti_fake_code|防伪码|string|N|CF6B2F6168420008|
|qr_code|二维码|string|N|demo|
|payer_register_no|购买方税务纳税人识别号。对企业开具电子发票时必填。
目前北京地区暂未开放对企业开具电子发票，若北京地区放开后，对于向企业开具的情况，付款方税务登记证号和名称也不能为空,注：根据国家税务总局公告2017年第16号公告，2017年7月1日起，增值税普通发票必须填写纳税人识别号，否则无法作为企业内部报销凭证。|varchar(20)|N|支持15、17、18、20位数字及字母，字母需小写。|
|payer_name|发票抬头名称|string(100)|N|开票成功必传|
|user_phone|消费者手机号|string(20)|Y| |
|payer_phone|购买方电话|string(16)|N| |
|payer_mobile|购买方电话(兼容旧接入系统，与payer_phone值相同)|string(16)|N| |
|reflect|反射参数|string(64)|N|任何调用者希望原样返回的信息，可以用于关联商户ERP系统的订单或记录附加订单内容, 比如 { "tips": "200" }|

- 回调参数示例： 

```javascript
{
    "terminal_sn":"2200000001",
    "task_sn":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "client_sn": "22000000012",
    "client_task_sn": "22000009989",
    "finish_time": "1492506702864",
    "invoice_date": "2017-01-12",
    "invoice_code": "150003528888",
    "invoice_no": "50877603",
    "anti_fake_code": "CF6B2F6168420008",
    "invoice_amount": "117000",
    "file_path":"demo",
    "payer_name": "发票抬头",
    "payer_mobile": "0512-55553333",
    "payer_phone": "0512-55553333",
    "user_phone:": "18268888888"
    "payer_email":"123@qq.com",
    "payer_register_no": "9133010060913454XP",
    "invoice_type":"0",
    "reflect":reflect_struct
}

```

- 回调返回说明：

 商户收到通知后应返回如下结果; 如果收钱吧未收到"成功"结果，或者没有调通回调接口。 不会重试，但开发者可以通过调用 query 接口进行自行补偿

- 回调返回示例：

```javascript
{
    "result_code": "200",
    "biz_response": {
        "result_code": "SUCCESS",
        "data": "接收开票信息成功"
    }
}
```


