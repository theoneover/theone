# theone
#phpstudy

<?php
namespace Home\Controller;
use Think\Controller;
class IndexController extends Controller {
	private $token = 'weixin';//token
	private $appID = 'wx6b6ad55cf189788a';
	private $appsecret = '684bdbb6d7ef98f3ed4d0d87614da654';
	private $header = array("apikey:be012c15889035efc7303caa78ba5358",);
	//接入
    public function index(){
    	if($this->checkSignature() && $_GET['echostr']){
    		echo $_GET['echostr'];
    	}else{
    		$this->response();
            //$this->diyMenu();
            $this->lunbo();
            $this->lists();
            $this->display();
    	}
    }

     public function lunbo() {
        $lists = M('Product')->where('status=1')->order('create_time')->limit(0, 3)->select();
        $this->assign('lunbo', $lists); // 赋值数据集lunbo
    }

    public function lists() {
        $lists = M('Product')->order('id asc')->limit(0, 4)->select(); //查询product  "id asc"
        $this->assign('lists', $lists);

        $lists_new = M('Product')->order('create_time desc')->limit(0, 4)->select(); //查询product  "id asc"
        $this->assign('lists_new', $lists_new);

        $lists_top = M('Product')->order('id')->limit(3, 4)->select(); //查询product  "id asc"
        $this->assign('lists_top', $lists_top);
    }


    private function checkSignature()
	{
	    $signature = $_GET["signature"];
	    $timestamp = $_GET["timestamp"];
	    $nonce = $_GET["nonce"];	
	        		
		$token = $this->token;
		$tmpArr = array($token, $timestamp, $nonce);
		sort($tmpArr, SORT_STRING);
		$tmpStr = implode( $tmpArr );
		$tmpStr = sha1( $tmpStr );
		
		if( $tmpStr == $signature ){
			return true;
		}else{
			return false;
		}
	}

	//根据事件 回复相应消息
	public function response(){
		/*
		<xml>
		<ToUserName><![CDATA[toUser]]></ToUserName>
		<FromUserName><![CDATA[FromUser]]></FromUserName>
		<CreateTime>123456789</CreateTime>
		<MsgType><![CDATA[event]]></MsgType>
		<Event><![CDATA[subscribe]]></Event>
		</xml>
		 */
		$postXml = $GLOBALS['HTTP_RAW_POST_DATA'];
		$postObj = simplexml_load_string($postXml);
		if($postObj->MsgType == 'event'){
			if($postObj->Event == 'subscribe'){
				$Content = "感谢您关注【请做我一生一世】" . "\n" . "微信号：请做我一生一世" . "\n" . "目前平台功能如下：" . "\n" . "【1】 查天气，如输入：武汉" . "\n" . "【2】 看新闻，如输入：tuwen" . "\n" . "更多内容，<a href='http://www.soso.com/'>戳我</a>";
			}elseif($postObj->Event == 'CLICK'){
				$Content = $postObj->Event;
			    if($postObj->EventKey == 'tianqi'){
			        $Content = '你点击的是自己的城市';
			    }
			}elseif($postObj->Event == 'SCAN'){
				$Content = $postObj->Ticket;
			}
		}elseif($postObj->MsgType == 'text'){
			switch($postObj->Content){
				case '天气':
					$Content = '今天挺凉快！';
					break;
				case 1:
					$Content = 'token是'.$this->getToken();
					break;
				case '2':
					$Content = 'jsApiTicket是'.$this->getJsApiTicket();
					break;
				case '3':
					$Content = "<a href='http://www.baidu.com'>百度一下</a>";
					break;
				case '4':
					$Content = "<a href='http://dddppp.free.wtb-cdn.pw/index.php/Home/Index/jssdk'>百度一下</a>";
					break;
                case '你好':
                    $Content = '你好！';
                    break;
                case '百度':
                    $Content = "<a href='http://www.baidu.com'>百度一下</a>";
                    break;
                 case '搜索':
                    $Content = "<a href='http://www.soso.com/'>百度一下</a>";
                    break;
                 case '我恨你':
                    $Content = '我爱你！';
                    break;
                 case '去死':
                    $Content = '我爱你！';
                    break;
                case '呵呵':
                    $Content ="<a href='http://www.soso.com/'>戳我</a>" ;
                    break;
				case 'tuwen':
					$ToUserName = $postObj->FromUserName;
					$FromUserName = $postObj->ToUserName;
					$CreateTime = time();
					$MsgType = 'news';
					$Title = '网易';
					$Description = '人人都玩不玩才怪';
					$PicUrl = "http://img6.cache.netease.com/photo/0001/2016-09-19/C1C4P8LT3R710001.jpg";
					$Url = "http://www.163.com/";
					$template = "<xml>
								<ToUserName><![CDATA[%s]]></ToUserName>
								<FromUserName><![CDATA[%s]]></FromUserName>
								<CreateTime>%s</CreateTime>
								<MsgType><![CDATA[%s]]></MsgType>
								<ArticleCount>1</ArticleCount>
								<Articles>
								<item>
								<Title><![CDATA[%s]]></Title>
								<Description><![CDATA[%s]]></Description>
								<PicUrl><![CDATA[%s]]></PicUrl>
								<Url><![CDATA[%s]]></Url>
								</item>
								</Articles>
								</xml> ";
					$info = sprintf($template,$ToUserName,$FromUserName,$CreateTime,$MsgType,$Title,$Description,$PicUrl,$Url);
					echo $info;
					break;
				default:
					$Content = $this->weather($postObj->Content);
					break;
			}
		}
		echo $this->textPush($postObj,$Content);
	}

	public function textPush($postObj,$Content){
		$ToUserName = $postObj->FromUserName;
		$FromUserName = $postObj->ToUserName;
		$CreateTime = time();
		$MsgType = 'text';
		$template = "<xml>
					<ToUserName><![CDATA[%s]]></ToUserName>
					<FromUserName><![CDATA[%s]]></FromUserName>
					<CreateTime>%s</CreateTime>
					<MsgType><![CDATA[%s]]></MsgType>
					<Content><![CDATA[%s]]></Content>
					</xml>";
		$info = sprintf($template,$ToUserName,$FromUserName,$CreateTime,$MsgType,$Content);
		return $info;
	}

	//自定义菜单
	public function diyMenu(){
		$token = $this->getToken();
		$url = "https://api.weixin.qq.com/cgi-bin/menu/create?access_token=".$token;
		$menu = array(
				'button'=>array(
					array(
						'type'=>'click',
						'name'=>'天气',
						'key' =>'tianqi',
						),
					array(
						'name'=>'菜单',
						'sub_button'=>array(
								array(
										'type'=>'view',
										'name'=>'搜索',
										'url' =>'http://www.soso.com/',
									),
								array(
										'type'=>'location_select',
										'name'=>'发送位置',
										'key' =>'rselfmenu_2_0',
										'sub_button'=>array(),
									),
								array(
										'type'=>'pic_weixin',
										'name'=>'微信相册发图',
										'key' =>'rselfmenu_1_2',
										'sub_button'=>array(),
									),
							),
						),
				),
			);
		$menu = $this->json_encode($menu);
        dump($this->curlSub($url, $menu));
	}

	//删除自定义菜单
    public function delMenu()
    {
        $token = $this->getToken();
        $url = 'https://api.weixin.qq.com/cgi-bin/menu/delete?access_token=' . $token;
        $res = $this->curlSub($url);
        dump($res);
    }

    //页面授权
    public function getBaseInfo(){
    	$redirect_uri = urlencode('http://dddppp.free.yz8888.cn/index.php/Home/Index/getBaseInfoUri');
    	$url = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid='.$this->appID.'&redirect_uri='.$redirect_uri.'&response_type=code&scope=snsapi_userinfo&state=12345#wechat_redirect';
    	// echo $url;die;
    	header('location:'.$url);
    }
    public function getBaseInfoUri(){
    	$code = $_GET['code'];
    	$url = 'https://api.weixin.qq.com/sns/oauth2/access_token?appid='.$this->appID.'&secret='.$this->appsecret.'&code='.$code.'&grant_type=authorization_code';
    	$res = $this->curlSub($url);
    	$res = json_decode($res,true);

    	$detailUrl = 'https://api.weixin.qq.com/sns/userinfo?access_token='.$res['access_token'].'&openid='.$res['openid'].'&lang=zh_CN';
    	$result = $this->curlSub($detailUrl);
    	$result = json_decode($result,true);
    	dump($result);
    }
    
    //天气查询
    public function weather($city){
        $url = 'http://apis.baidu.com/apistore/weatherservice/cityname?cityname='.$city;
        $res = json_decode($this->curlSub($url,'',$this->header),true);
        $data = $res['retData'];
        if(!$data){
            return '无相关信息';
        }
        return "{$data['city']}：{$data['weather']}\r气温：{$data['temp']}\r最低气温：{$data['l_tmp']}\r最高气温：{$data['h_tmp']}\r风向：{$data['WD']},风力：{$data['WS']}\r日出时间：{$data['sunrise']}\r日落时间：{$data['sunset']}";
    }

    //临时二维码
    public function QrCode(){
    	$url = "https://api.weixin.qq.com/cgi-bin/qrcode/create?access_token=".$this->getToken();
    	$json = '{"action_name": "QR_LIMIT_SCENE", "action_info": {"scene": {"scene_id": 123}}}';
    	$Qr = json_decode($this->curlSub($url,$json),true);
    	$QrUrl = 'https://mp.weixin.qq.com/cgi-bin/showqrcode?ticket='.urlencode($Qr['ticket']);
    	// $res = $this->curlSub($QrUrl);
    	header('location:'.$QrUrl);
    }
    
	//获取token
	public function getToken(){
		if(!S('access_token')){
			$url = 'https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid='.$this->appID.'&secret='.$this->appsecret;
			$arr = json_decode($this->curlSub($url),true);
			S('access_token',$arr['access_token'],7200);
		}
		return S('access_token');
	}

	//curl提交
	public function curlSub($url,$post = '',$header = ''){
		//初始化curl
		$ch = curl_init();
		//设置curl参数
		if (stripos($url, "https://") !== FALSE) {
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
            curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
        }
        if($post != ''){
        	curl_setopt($ch, CURLOPT_POST, 1);
        	curl_setopt($ch, CURLOPT_POSTFIELDS, $post);
        }
        if($header != ''){
            curl_setopt($ch, CURLOPT_HTTPHEADER  , $header);
        }
		curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        //3.采集
        $result = curl_exec($ch);
        if (curl_error($ch)) {
            dump(curl_error($ch) . curl_errno($ch));
        }
        //4.关闭
        curl_close($ch);
        return $result;
	}

	 /**
     * 微信api不支持中文转义的json结构
     * @param array $arr
     */
    static function json_encode($arr)
    {
        $parts = array();
        $is_list = false;
        //Find out if the given array is a numerical array
        $keys = array_keys($arr);
        $max_length = count($arr) - 1;
        if (($keys [0] === 0) && ($keys [$max_length] === $max_length)) { //See if the first key is 0 and last key is length - 1
            $is_list = true;
            for ($i = 0; $i < count($keys); $i++) { //See if each key correspondes to its position
                if ($i != $keys [$i]) { //A key fails at position check.
                    $is_list = false; //It is an associative array.
                    break;
                }
            }
        }
        foreach ($arr as $key => $value) {
            if (is_array($value)) { //Custom handling for arrays
                if ($is_list)
                    $parts [] = self::json_encode($value); /* :RECURSION: */
                else
                    $parts [] = '"' . $key . '":' . self::json_encode($value); /* :RECURSION: */
            } else {
                $str = '';
                if (!$is_list)
                    $str = '"' . $key . '":';
                //Custom handling for multiple data types
                if (is_numeric($value) && $value < 2000000000)
                    $str .= $value; //Numbers
                elseif ($value === false)
                    $str .= 'false'; //The booleans
                elseif ($value === true)
                    $str .= 'true';
                else
                    $str .= '"' . addslashes($value) . '"'; //All other things
                // :TODO: Is there any more datatype we should be in the lookout for? (Object?)
                $parts [] = $str;
            }
        }
        $json = implode(',', $parts);
        if ($is_list)
            return '[' . $json . ']'; //Return numerical JSON
        return '{' . $json . '}'; //Return associative JSON
    }
    
    public function get_real_ip(){
        $ip=false;
        if(!empty($_SERVER['HTTP_CLIENT_IP'])){
            $ip=$_SERVER['HTTP_CLIENT_IP'];
        }
        if(!empty($_SERVER['HTTP_X_FORWARDED_FOR'])){
            $ips=explode (', ', $_SERVER['HTTP_X_FORWARDED_FOR']);
            if($ip){ array_unshift($ips, $ip); $ip=FALSE; }
            for ($i=0; $i < count($ips); $i++){
                if(!eregi ('^(10│172.16│192.168).', $ips[$i])){
                    $ip=$ips[$i];
                    break;
                }
            }
        }
        return $ip ? $ip : $_SERVER['REMOTE_ADDR'];
    }

	/*public function ipToCity(){
		$ip = $this->get_real_ip();
		$url = 'http://apis.baidu.com/chazhao/ipsearch/ipsearch?ip='.$ip;
		$res = $this->curlSub($url,'',$this->header);
		return $city = json_decode($res,true)['data']['city'];
	}*/

	public function jssdk(){
		$ticket = $this->getJsApiTicket();
		$nonceStr = $this->randStr();
		$timestamp = time();
		$protocol = (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off' || $_SERVER['SERVER_PORT'] == 443) ? "https://" : "http://";
    	$url = "$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]";
    	$string = "jsapi_ticket=$ticket&noncestr=$nonceStr&timestamp=$timestamp&url=$url";
		$signature = sha1($string);
		dump($url);
		dump($nonceStr);
		dump($ticket);
		dump($timestamp);
		dump($string);
		dump($signature);
		$this->assign('appId',$this->appID);
		$this->assign('timestamp',time());
		$this->assign('nonceStr',$this->randStr());
		$this->assign('signature',$signature);
		$this->display();
	}

	/*public function getJsApiTicket(){
		if(!S('jsApiTicket')){
			$token = $this->getToken();
			$url = 'https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token='.$token.'&type=jsapi';
			S('jsApiTicket',(json_decode($this->curlSub($url),true)['ticket']),7000);
		}
		return S('jsApiTicket');
	}*/

	public function randStr($length = 16){
		$chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890';
		$str = '';
		for($i=0;$i<$length;$i++){
			$str .= $chars[rand(0,$length)];
		}
		return $str;
	}

	public function test(){
		dump(U('Home/Index/index'));
		dump($this->appId);
		dump($_SERVER['HTTP_HOST']);
		dump($_SERVER["REQUEST_URI"]);
	}
}
