# Thinkphp3.2-WechatPersonInfo
thinkphp3.2结合微信公众号的授权获取微信账号信息
### 注意：你需要先了解oauth2协议，并且你的公众号支持网页授权服务，改例子是用测试号
### 第一步、编写入口方法（首先通过接入微信的oauth2连接协议接口，再进入回调地址redirect_uri）
```php
public function person(){
  echo "<script language='javascript'>"; 
  echo " location='https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx3720ab9700b32c13&redirect_uri=http://www.test.com/test/index.php/Home/wechat/oauth&response_type=code&scope=snsapi_userinfo&state=1&from=singlemessage#wechat_redirect ';"; 
  echo "</script>";
}
```
### 第二步、微信会返回一个code值，然后再请求微信的另一个oauth2接口，可以获取access_token和openid
```php
 public function oauth2(){
$code = $_GET["code"];
$appid = "你的appid";
$appsecret = "你的appsecret";
//以下为获取全局access_token
$url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=$this->appid&secret=$this->appsecret&code=$code&grant_type=authorization_code";//使用code获取access_token
//$url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=$appid&secret=$appsecret";//这里是获取全局access_token
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);  // 设置你需要抓取的URL
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);  //这两句用于验证第三方服务器与微信服务器的安全性
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);  //这两句用于验证第三方服务器与微信服务器的安全性
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  // 设置cURL 参数，要求结果保存到字符串中还是输出到屏幕上。1是不输出在页面上
$output = curl_exec($ch);   //得到的access_token赋值给$output
curl_close($ch);
$jsoninfo = json_decode($output, true);
$access_token = $jsoninfo["access_token"];
$openid = $jsoninfo["openid"];
$this->getInfo($access_token,$openid);

    }
```
### 第三步、调用getinfo方法，通过上面获得的openid和accesss_token获取详细信息
```php
public function getInfo($access_token,$openid){
  $url = "https://api.weixin.qq.com/sns/userinfo?access_token=$access_token&openid=$openid&lang=zh_CN";
  $curl = curl_init();
  curl_setopt($curl, CURLOPT_URL, $url);
  curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, FALSE);
  curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, FALSE);
  curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
  $data = curl_exec($curl);
  if (curl_errno($curl)) {return 'ERROR '.curl_error($curl);}
  curl_close($curl);
  $output = $data;
  $jsoninfo = json_decode($output);
  $_SESSION['headimgurl'] = $jsoninfo -> headimgurl;
  $_SESSION['openid'] = $jsoninfo -> openid;
  $_SESSION['nickname'] = $jsoninfo -> nickname;
  $_SESSION['sex'] = $jsoninfo -> sex;
  $_SESSION['language'] = $jsoninfo -> language;
  $_SESSION['country'] = $jsoninfo -> country;
  $_SESSION['subscribe'] = $jsoninfo -> subscribe;//是否关注公众号

  echo "<script language='javascript'>";
  echo " location='http://www.test.com/Home/index/mydemo';";
  echo "</script>";
}
```
### 第三步、视图渲染
```php
public function personinfommydemo(){
  $this->assign('info',$session);
  $this->display('personinfomydemo');
}
```
### 第四步、视图接收数据
```php
    <p>微信头像：</p> <img alt="正在加载..."  style="width:60%;height:20%;border-radius: 50%;"  src="{$info['headimgurl']}">
    <p>微信名称：{$info['nickname']}</p>
    <p>微信地址：{$info['country']}{$info['province']}{$info['city']}</p>
    <p>微信性别：{$info['sex']}</p>
    <p>微信openid：{$info['openid']}</p>
```
