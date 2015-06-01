基于linkNum/textNum比例的网页去噪

## 1. 介绍： ##

### 1) xpspider ###
对于一个非hub页，除了正文之外，在周边通常存在一些链接或者广告等"噪声"信息。通过编写正则表达式可以方便且准确地对正文进行抽取(例如工具：[xpspider](http://code.google.com/p/xpspider/))，但是需要具备正则表达式知识。

### 2) 本工具采用了一种基于统计学的新方法： ###
> 首先对获取的HTML代码创建DOM树；

> 然后深度优先遍历DOM树，对每个结点统计其包含的链接数目linkNum和包含的非连接的文字个数textNum；

> 计算linkNum和textNum的比值，并将该比值和一个设定的值ratio进行比较:
如果大于ratio，则从DOM树中删除该结点，反之则保留；

> 由于一般情况下正文部分和噪声部分的linkNum/textNum值相差较大，如果阈值ratio设置合理，可以有效地从DOM树中删除噪声结点，仅剩余网页的主体部分。

> 该方法不需要编写规则，只需要多测试几个页面，确定合适的阈值ratio即可;

### 3) 类HTMLExtractor是对如上方法的实现，具体请下载； ###

## 2. 使用: ##
```
require_once("HTMLExtractor.php");

//1. 创建一个HTMLExtractor对象，可以为构造函数传递ratio值；如果不设置，默认为0.05。
//阈值ratio设置得越低，被删除的结点越多；
$x=new HTMLExtractor(0.036);


//2. 设置参数(可以不设置，各个选项均有默认值，详见HTMLExtractor.php)：

//setIngoreTags方法可以设置直接被删除的tag结点；参数$ignoreTags必须是一个数组，默认为：$ignoreTags=array("script","meta","link","object","param","noscript","style","form","button","meta","input","select","iframe","embed","hr","img");
//$x->setIgnoreTags($ignoreTags);

//setRatio方法可以设置阈值ratio，如果没有在HTMLExtractor构造函数传递，也可以使用该方法设置；
//$x->setRatio($ratio);

//setFilterWords方法可以设置过滤的噪声词，即如果一个文本结点包含有噪声词，该文本结点将被删除，通常用于过滤一些微小的、不易使用ratio阈值限制的噪声结点；$filterWords必须是一个数组而且必须为utf-8类型，如果是其他类型需要提前转换。
$filterWords=array("不支持Flash");
for($i=0;$i<count($filterWords);$i++)
{
	$filterWords[$i]=iconv("gb2312","utf-8//IGNORE",$filterWords[$i]);
}
$x->setFilterWords($filterWords);


//3. 去噪：HTMLExtractor类提供了两种方式获取HTML源代码：

//1) 从文件获取：
$filename="src.shtml";
if(!$x->extractFromHTMLFile($filename))
{
	print "Title: ".$x->getTitle()."\n";
	print "Body: ".$x->getBody()."\n";
	//print "Body: ".$x->getTextBody()."\n";
}

//2) 从url获取：
if(!$x->extractFromUrl("http://news.bbc.co.uk/2/hi/south_asia/8157795.stm"))
{
	print "Title: ".$x->getTitle()."\n";
	print "Body: ".$x->getBody()."\n";
	//print "Body: ".$x->getTextBody()."\n";
}

//去噪后的结果，可以通过getTitle, getBody和getTextBody获得：
//getTitle获取当前网页的标题信息；
//getBody获取当前网页的主题内容信息，以HTML格式；
//getTextBody获取当前网页的主题内容信息，纯文本格式不包含HTML tag；

```