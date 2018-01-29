# Smart-Y
解けず。
WEBの問題。72人が解答。
問題文は以下。
> Last year, a nerd destroyed the system of Robot City by using some evident flaws. It seems that the system has changed and is not as evident to break now.

>http://smart-y.teaser.insomnihack.ch

google翻訳を使うとこんな感じ。
> 昨年、オタクはRobot Cityのシステムをいくつかの明白な欠陥を使って破壊しました。 システムが変更されたように見えて、今すぐ中断することは明らかではありません。

問題文に記載されているURLに飛ぶと、以下のようなページが現れる。

![Alt text](smarty.bmp)

DEBUGと書かれているリンクを押すと、以下のコードが現れる。

```
<?php 

if(isset($_GET['hl'])){ highlight_file(__FILE__); exit; } 
include_once('./smarty/libs/Smarty.class.php'); 
define('SMARTY_COMPILE_DIR','/tmp/templates_c'); 
define('SMARTY_CACHE_DIR','/tmp/cache'); 
  
  
class news extends Smarty_Resource_Custom 
{ 
    protected function fetch($name,&$source,&$mtime) 
    { 
        $template = "The news system is in maintenance. Please wait a year. <a href='/console.php?hl'>".htmlspecialchars("<<<DEBUG>>>")."</a>"; 
        $source = $template; 
        $mtime = time(); 
    } 
} 
  
// Smarty configuration 
$smarty = new Smarty(); 
$my_security_policy = new Smarty_Security($smarty); 
$my_security_policy->php_functions = null; 
$my_security_policy->php_handling = Smarty::PHP_REMOVE; 
$my_security_policy->modifiers = array(); 
$smarty->enableSecurity($my_security_policy); 
$smarty->setCacheDir(SMARTY_CACHE_DIR); 
$smarty->setCompileDir(SMARTY_COMPILE_DIR); 


$smarty->registerResource('news',new news); 
$smarty->display('news:'.(isset($_GET['id']) ? $_GET['id'] : ''));  
```

以上が問題の概要。





