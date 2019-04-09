
<?php
# 接入指南


------
##签名规则

1）筛选参数：剔除sign字段
2）排序：将筛选得到的参数按照第一个字符的键值ASCII码递增排序（字母升序排序），如果遇到相同字符则按照第二个字符的键值ASCII码递增排序，以此类推。
3）拼接：将排序后的参数与其对应值，组合成"参数=参数值"的格式，并且把这些参数用&字符连接起来，最后再拼接"&key=$key",此时生成的字符串为待签名字符串。（即key1=value1&key2=value2…）拼接而成，空值不参与签名.
4）签名：用MD5算法对生成的待签名字符串逬行加密，获得签名
5）签名原始串中，字段名和字段值都采用原始值，不进行urlEncode。

*(注意 如果是发起支付请求 跟 查询订单请求的,请用支付秘钥参与验签  如果是本站通知你方的回调方法时 请用回调秘钥参与签名 )

##发起支付

接口地址： http://zhifu.com
请求方式  POST

请求参数 
| 参数名         | 说明                                  |  是否必须  |     类型   |
| --------      |--------                               | --------  |    ------  |
| order_id      |商户订单号                              |  是       |     string |
| pay_amount    |充值金额 (分)                           |   是      |     int    |
| userId        |玩家id                                  |   是      |    int     |
| timestamp    |订单时间戳                            |   是      |     string |
| notify_url    |异步通知地址                            |   是      |     string |
| return_url    |同步通知地址                            |   否      |     string |
| channel       |渠道号 (格式:chang-aipayh5-pay)         |   是      |     string |
| token         |签名字符串                              |   是      |     string |



##异步通知接口 
 当第三方异步通知本站时 经过相关处理后 本站会向你们提供的回调地址 以post方式 带上请求json数据 你们直接用post方式接收并json_decode 将得到一个数组 如下  当code为字符串 '0000'时 说明支付成功
  
为了接口的安全性 请去掉sign 字段 其他字段按照字典排序 再拼上回调秘钥 再md5加密
 例子: 
 "order_id=201039990565&pay_amount=1000&plat_oid=20170305655&222560ed1794bbbc3c2fe06d848252ad"

json_decode 得到的数组如下:
    array(
          'code'=>'0000',
          'msg'=>'ok',
          array(
               'order_id'=>'201039990565',        //用户订单   string
               'plat_id'=>'20170305655',       //平台订单    string
               'pay_amount'=>1000,              //实际付款金额 int
               'sign'=>'222560ed1794bbbc3c2fe06d848252ad',     //签名 string
         ) 
    );


| 参数名           | 说明                             |  是否必须  |
| --------        | --------                         | --------   |
| code            | 状态码                            |   是     |
| msg             |  交易状态 ok:成功，error：失败     |   是     |
| order_id        |商户订单号                         |  是      |
| plat_id         |平台订单号                         |   是     |
| pay_amount      |充值金额 单位是 分                  |   是     |



请在验签通过后 处理完业务逻辑后 返回 {"error":"0000","msg":"ok"}
若处理失败 返回 {"error":"2000","msg":"error"}





##同步通知接口
接口通过浏览器地址跳转的方式调用 发起支付中的 return_url 地址，参数列表如下
| 参数名        | 说明   |  是否必须  |
| --------   | --------   | --------   |
| merchant_id     | 商户id |   是     |
| trade_status     | 交易状态 SUCESS:成功，FAIL：失败 |   是     |
| trade_no |第三方订单号|   是   |
| out_trade_no |商户订单号|   是   |
| total_amount |充值金额 单位是 分|   是   |
| channel_code |充值渠道|   是   |
| version |接口版本 填：1.0|是      |
| timestamp |当前时间戳 s|   是   |
| sign_type |签名类型 填写：MD5|   是   |

##交易状态查询接口
在需要确认交易的真实状态信息的时候我们可以调用这个接口

接口地址：域名根目录
特殊说明：发起支付需要在请求头添加参数 method 值为 query

需要参数据如下,
| 参数名        | 说明   |  是否必须  |
| --------   | --------   | --------   |
| merchant_id     | 商户id |   是     |
| trade_no |第三方订单号|   是   |
| version |接口版本 填：1.0|是      |
| timestamp |当前时间戳 s|   是   |
| sign_type |签名类型 填写：MD5|   是   |

返回参数据如下
| 参数名        | 说明   |  是否必须  |
| --------   | --------   | --------   |
| merchant_id     | 商户id |   是     |
| trade_status     | 交易状态 SUCESS:成功，FAIL：失败 |   是     |
| trade_no |第三方订单号|   是   |
| out_trade_no |商户订单号|   是   |
| total_amount |充值金额 单位是 分|   是   |
| channel_code |充值渠道|   是   |
| version |接口版本 填：1.0|是      |
| timestamp |当前时间戳 s|   是   |
| sign_type |签名类型 填写：MD5|   是   |



##系统当前支持的支付方式以及金额

这个接口是为了解决，支付方式与支付金额 固定的问题

接口地址：域名根目录
特殊说明：发起支付需要在请求头添加参数 method 值为 config

返回值，返回的是一个支付方式组成的数组，如果支持多种支付方式则会返回多个元素。
pay_type：
     * ali_h5支付宝H5
     * ali_qr支付宝扫码
     * wx_h5微信H5
     * wx_qr微信扫码
[
    {
        "pay_type":"ali_h5",//支付方式
        "amount":{//金额限制条件
            "max":10000000,//最高多少 肯定会返回
            "min":10,//最低多少 也会返回
            "list":[ //支持的金额列表 如果 没有限制则会为空 单位是元
                100,
                200,
                300,
                400
            ]
        }
    }
]