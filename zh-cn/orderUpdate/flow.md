# 接入流程

## 对接过程的业务流程

不同于接口开票方案，使用订单推送方案的用户无需关系激活、签到等行为，收钱吧将给出已激活的终端号，商户只需根据对应的终端号进行订单上传即可使用开票业务。

## 接入说明
 > * 第一步: 商家提交商户资料，获取门店及终端信息，并选择对应开票方案
 > * 第二部: 接入[订单同步服务](interface.md)，通过接口上传订单作为开票信息
 > * 第三部: 根据[二维码生成规则](qrcode_guide.md)在小票或收据上打印二维码供消费者扫码开票