> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_38055667/article/details/103743085)

```
//加密(rc4)
function rc4encrypt($data,$pwd){
    $res = rc4($data,$pwd);
    return 'CB01'.toHex($res);
}
//解密(rc4)
function rc4decrypt($data,$pwd){
    $str = fromHex($data);
    return rc4($str,$pwd);
}
/**
 * [rc4 description]
 * @param  [type] $data [需加密的数据]
 * @param  [type] $pwd  [密钥]
 * @return [type]       [加密后为二进制，需要转换成16进制]
 */
function rc4($data, $pwd) {
    $key[]       = "";
    $box[]       = "";
    $pwd_length  = strlen($pwd);
    $data_length = strlen($data);
    $cipher      = '';
    for ($i = 0; $i < 256; $i++) {
        $key[$i] = ord($pwd[$i % $pwd_length]);
        $box[$i] = $i;
    }
    for ($j = $i = 0; $i < 256; $i++) {
        $j       = ($j + $box[$i] + $key[$i]) % 256;
        $tmp     = $box[$i];
        $box[$i] = $box[$j];
        $box[$j] = $tmp;
    }
    for ($a = $j = $i = 0; $i < $data_length; $i++) {
        $a       = ($a + 1) % 256;
        $j       = ($j + $box[$a]) % 256;
        $tmp     = $box[$a];
        $box[$a] = $box[$j];
        $box[$j] = $tmp;
        $k = $box[(($box[$a] + $box[$j]) % 256)];
        $cipher .= chr(ord($data[$i]) ^ $k);
    }
    return $cipher;
}
/**
 * 把数据转换成16进制
 * @param  [type]  $sa  [需要转换的数据]
 * @param  integer $len [数据长度]
 */
function toHex($sa , $len = 0){
    $buf = "";
    if( $len == 0 )
        $len = strlen($sa) ;
    for ($i = 0; $i < $len; $i++)
    {
        $val = dechex(ord($sa{$i}));  
        if(strlen($val)< 2) 
            $val = "0".$val;
            $buf .= $val;
    }
    return $buf;
}
/**
 * 把16进制转换成字符串
 * @param  [type] $sa [16进制数据]
 */
function fromHex($sa){
    $buf = "";
    $len = strlen($sa) ;
    for($i = 0; $i < $len; $i += 2){
        $val = chr(hexdec(substr($sa, $i, 2)));
        $buf .= $val;
    }
    return $buf;
}
```