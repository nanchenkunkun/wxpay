
#微信支付
**微信支付流程**
![](https://pay.weixin.qq.com/wiki/doc/api/img/chapter7_4_1.png)

		以下是支付场景的交互细节：
		
		（1）用户打开商户网页选购商品，发起支付，在网页通过JavaScript调用getBrandWCPayRequest接口，发起微信支付请求，用户进入支付流程。
		
		（2）用户成功支付点击完成按钮后，商户的前端会收到JavaScript的返回值。商户可直接跳转到支付成功的静态页面进行展示。
		
		（3）商户后台收到来自微信开放平台的支付成功回调通知，标志该笔订单支付成功。
#微信内H5调起支付
在微信浏览器里面打开H5网页中执行JS调起支付。接口输入输出数据格式为JSON。
注意：WeixinJSBridge内置对象在其他浏览器中无效。

1、网页端接口请求参数列表（参数需要重新进行签名计算，参与签名的参数为：appId、timeStamp、nonceStr、package、signType，参数区分大小写。）详见https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6

2.返回结果值说明

		 get_brand_wcpay_request:ok->支付成功
		 get_brand_wcpay_request:cancle->支付取消
		 get_brand_wcpay_request:fail->支付失败
		 缺少支付JSAPI缺少参数:total_fee->1.检查预支付会话标识prepay_id是否已经失效2.请求的appid与下单接口的appid是否一致
注：JS API的返回结果get_brand_wcpay_request:ok仅在用户成功完成支付时返回。由于前端交互复杂，get_brand_wcpay_request:cancel或者get_brand_wcpay_request:fail可以统一处理为用户遇到错误或者主动放弃，不必细化区分。

	function onBridgeReady(){
	   WeixinJSBridge.invoke(
	      'getBrandWCPayRequest', {
	         "appId":"",     //公众号名称，由商户传入     
	         "timeStamp":"",         //时间戳，自1970年以来的秒数     
	         "nonceStr":"e61463f8efa94090b1f366cccfbbb444", //随机串     
	         "package":"prepay_id=u802345jgfjsdfgsdg888",     
	         "signType":"MD5",         //微信签名方式：     
	         "paySign":"70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名 
	      },
	      function(res){
	      if(res.err_msg == "get_brand_wcpay_request:ok" ){
	      // 使用以上方式判断前端返回,微信团队郑重提示：
	            //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
	      } 
	   }); 
	}
	if (typeof WeixinJSBridge == "undefined"){
	   if( document.addEventListener ){
	       document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
	   }else if (document.attachEvent){
	       document.attachEvent('WeixinJSBridgeReady', onBridgeReady); 
	       document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
	   }
	}else{
	   onBridgeReady();
	}
##1.生成图文消息
		这个就是商家自定义的页面
		-->将图文消息展示给用户

##2.点击消息或者相关二维码在浏览器上打开商户H5页面
		-->网页内请求生成支付订单
##4.生成商户订单
	
1. 获取用户openid
 
		jsapi.php:
			$tools=new JsApiPay();
			$openId=$tools->GetOpenid();

		WxPay.JsApiPay.php:
			public function GetOpenid()
			{
				//通过code获得openid
				if (!isset($_GET['code'])){
					//触发微信返回code码
					$baseUrl = urlencode('http://'.$_SERVER['HTTP_HOST'].$_SERVER['REQUEST_URI'].$_SERVER['QUERY_STRING']);
					$url = $this->_CreateOauthUrlForCode($baseUrl);
					Header("Location: $url");
					exit();
				} else {
					//获取code码，以获取openid
				    $code = $_GET['code'];
					$openid = $this->getOpenidFromMp($code);
					return $openid;
				}
			}

     包装下单所需要的参数 

			$input = new WxPayUnifiedOrder();
			$input->SetBody("test");
			$input->SetAttach("test");
			$input->SetOut_trade_no("sdkphp".date("YmdHis"));
			$input->SetTotal_fee("1");
			$input->SetTime_start(date("YmdHis"));
			$input->SetTime_expire(date("YmdHis", time() + 60000));
			$input->SetGoods_tag("test");
			$input->SetNotify_url("http://dc.zzz001.com/php_sdk_v3.0.9/example/finally.php");
			$input->SetTrade_type("JSAPI");
			$input->SetSubMchId("1525372441");
			$input->SetOpenid($openId);
			$config = new WxPayConfig();
			$order = WxPayApi::unifiedOrder($config, $input);
			echo '<font color="#f00"><b>统一下单支付单信息</b></font><br/>';
			printf_info($order);
			$jsApiParameters = $tools->GetJsApiParameters($order);

			$order参数有预支付订单prepay_id



商户系统先调用该接口在微信支付服务后台生成预支付交易单，返回正确的预支付交易会话标识后再按Native、JSAPI、APP等不同场景生成交易串调起支付。

##5.调用统一下单API
		-->生成预支付订单
			-->返回预支付订单信息(prepay_id)

		客户主动发起支付请求:
		function jsApiCall()
			{
				WeixinJSBridge.invoke(
					'getBrandWCPayRequest',
					<?php echo $jsApiParameters; ?>,
					function(res){
						// WeixinJSBridge.log(res.err_msg);
						if(res.err_msg == "get_brand_wcpay_request:ok" ){
			  				window.location.assign("http://dc.zzz001.com/php_sdk_v3.0.9/example/finally.php");
			            } 
						
					}
				);
			}
		
			function callpay()
			{
				if (typeof WeixinJSBridge == "undefined"){
				    if( document.addEventListener ){
				        document.addEventListener('WeixinJSBridgeReady', jsApiCall, false);
				    }else if (document.attachEvent){
				        document.attachEvent('WeixinJSBridgeReady', jsApiCall); 
				        document.attachEvent('onWeixinJSBridgeReady', jsApiCall);
				    }
				}else{
				    jsApiCall();
				}
			}
			</script>
##6.生成JSAPI页面调用的支付参数并且签名
		-->返回支付参数（prepay_id,paySign)
##7.用户点击发起支付
		-->JSAPI接口请求支付
##8.检查参数合法性和授权域权限
		-->返回验证结果，并要求支付授权
			-->提示输入密码
##9.确认支付，输入密码
		-->提交支付授权
			-->验证授权
##10.异步通知商户支付结果
##11.告知微信通知处理结果
##12.返回支付结果，并发微信消息提示
		-->展示支付消息给用户
