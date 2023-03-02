### 图片裁切程序（ImageMagic）

关键步骤说明

图片路径规则：`http://{domain}/images/245x140/2023/1/70/{随机字符串}_{md5前5位}.jpg`

nginx 转发加入规则：访问图片文件 404 时 302转发至php `error_page 404 =302 /error.php;`

ImageMagic 命令 `@system("convert -geometry $photo_sizex$photo_size $photo_source $photo_target");`

创建文件夹：

```
复制function CreateDir($path){
//     $dir = split("/",$path);
    $dir = explode("/",$path);
    if(!is_dir($path)){
        $updir = "";
        for($i = 0; $i < count($dir); $i ++){
            $updir .= $dir[$i] . "/";
            if(!is_dir($updir)){
                mkdir($updir,0755);
            }
        }
    }
}

```

做小图:

```
复制/**
 * 参数为数组 key列表如下
 * photo_size 图片宽高的最大值
 * photo_source  图片原图
 * photo_target  目标图片
 */
function ph_resize_photo($photo_array=array()){
    $photo_size = $photo_array['photo_size'];
    $photo_source = $photo_array['photo_source'];
    $photo_target = $photo_array['photo_target'];
    //创建目标路径
    CreateDir(substr($photo_target,0,strrpos($photo_target,'/')));
    @system("convert -geometry $photo_sizex$photo_size $photo_source $photo_target");
}

```

业务代码示例：

```
复制preg_match_all('/\/(.*)\/(.*)\/(.*)\/(.*)\/(.*)\/(.*)/i',$_SERVER["REQUEST_URI"],$out);

if(@$out[1][0]=='images'){
    include_once('./loader.php');
    //$alow_array = split(',',$PHOTO_SETTING['width']);
    if(!in_array(@$out[2][0],$PHOTO_WIDTH)){
        include($DOCUMENT_ROOT.'/common/error.html');
        exit;
    }

    $location = strpos(@$out[6][0],'_');
    $md5_str = substr(@$out[6][0],$location+1);
    $md5_str = strtok($md5_str,'.');
    if($md5_str != substr(md5(substr(@$out[6][0],0,$location).@$out[3][0].@$out[4][0]),0,5)){
        include($DOCUMENT_ROOT.'/common/error.html');
        exit;
    }

    $phot_orc = $PHOTO_PATH.'source/'.@$out[3][0].'/'.@$out[4][0].'/'.@$out[5][0].'/'.@$out[6][0];
    if(!file_exists($phot_orc)){
        include($DOCUMENT_ROOT.'/common/error.html');
        exit;
    }

    $photo_arr = array(
        'photo_size' => @$out[2][0],
        'photo_source' => $phot_orc,
        'photo_target' => $PHOTO_PATH.@$out[2][0].'/'.@$out[3][0].'/'.@$out[4][0].'/'.@$out[5][0].'/'.@$out[6][0]
    );

    ph_resize_photo($photo_arr);

    header('location: '.$_SERVER["REQUEST_URI"].'?'.rand());
    exit();
}else{
    include($DOCUMENT_ROOT.'/common/error.html');
}

```