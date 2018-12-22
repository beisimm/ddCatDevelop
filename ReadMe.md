# 多多猫插件开发记录
## 多多猫APP
多多猫是一款比较另类的app它以插件形式提供开放平台，只要熟悉HTML,CSS,JavaScript，即可成为插件开发者。
插件有漫画，轻小说，视频，图片，资讯等各种分类，分别提供不同的内容。

如果你没听过这款软件，可以试着用一下（并没有利益相关），这样对于插件的开发和使用，也有个基本概念。
多多猫插件，可以看作是一个爬虫，运行在多多猫这个容器里，数据的解析和渲染由容器支持，而我们只需要编写函数告诉容器，应该怎样获取数据即可。

## 调试

1. 手机下载手机下载多多猫
2. 电脑和手机在同一wifi下(便于在手机上调试插件)
3. 电脑安装一个本地服务器，比如XAMPP（下面的教程，就以XAMPP为例）

在xampp安装目录下的htdocs文件夹下新建一个文件，命名MyProject.sited.html(注意：插件的后缀名均为sited, html为未上传的明文代码, 多多猫也可以识别)

手机打开多多猫，在搜索框中输入:192.168.191.3:8080/MyProject.sited.html(前面写apache的Ip和端口号)，于是多多猫就安装了我们本地的插件

[调试记录].开启开发者模式：进入安卓多多猫app的设置-高级设置。长按标题的“高级设置”几个字，最底部多出一次性出现的开关。在多多猫目录里/sdcard/Android/data/org.noear.ddcat/files可以找到log运行日志、error日志等。在插件的function里可以写print(“字符串”)、print(函数)，在app相应节点执行时候会显示出字符串或函数值，在刚才多多猫目录里可以找到print过的日志。

``` html
<?xml version="1.0" encoding="utf-8"?>
<sited ver="10" debug="1" engine="27" schema="1">  //var 版本号 debug 是否开启debug模式 engine 引擎版本最高35
	<meta>
	<ua></ua>
	<title>57漫画手机版</title>  //插件显示名称
	<intro>57漫画手机版是一个看漫画的SiteD插件</intro>
	<author>鹿财</author>
	<url>http://m.57mh.com</url>
	<expr>\.57mh\.com</expr>
	<encode>utf-8</encode>  //编码格式
	</meta>
	<main dtype="1" durl="http://m.57mh.com">  // dtype:1:漫画；2:轻小说；3:动画；4:图片（无目录）；5:周边；6:资讯；7:视频（无目录）
		<home>
			<hots cache="1d" title="今日推荐" method="get" parse="hots_parse" url="http://m.57mh.com" />  // parse会调用下方函数来解析
			<tags title="标签">
				<item title="最近更新" url="http://m.57mh.com/latest" />
				<item title="日行榜" url="http://m.57mh.com/rank" />
			</tags>
		</home>
		<tag cache="1d" method="get" parse="tag_parse" />  // cache(1=永久；1d=1 天；1m=1 分钟；0=不缓存)
		<book cache="1d" method="get" parse="book_parse" />  // 书本信息目录作者等method请求方式
		<section cache="1" method="get" parseUrl='section_parseUrl' parse="section_parse" header="cookie;referer" />
	</main>
	<script>
		<require>
			<item url = "http://sited.noear.org/addin/js/cheerio.js" lib = "cheerio" />  //引用库
		</require>
		<code>
			<![CDATA[
				var host = 'http://m.57mh.com';
				function urla(u) {  // url补全
					if (u.indexOf("http") < 0) {
						if (u.substr(0, 1) != '/')
							u = host + '/' + u;
						else
							u = host + u;
					}
					return encodeURI(u);
				}

				function hots_parse(url, html) {  // 首页地址解析
					var $ = cheerio.load(html);
					var list = [];
					var books = $('#data_list li');


					books.each(function() {
						var book = $(this);
						var name = book.find('h3').text();
						var url = book.children('a').attr('href');
						var logo = book.find('img').attr('src');

						list.push({
							name: name,
							url: host + url,
							logo: logo
						});
					});
					return JSON.stringify(list);

				}

				

				function book_parse(url, html) {  // 书本解析, 作者, 封面, 信息, 更新日期等
					var $ = cheerio.load(html);
					var data = {};
					
					data.name = '';
					data.author = '';
					data.logo = '';
					data.intro = $('#bookIntro p').text();
					data.updateTime = '';
					data.isSectionsAsc = 0;
					data.sections = [];
					data.author = '';
					$('#chapterList a').each(function(){
						var al = $(this);
						
						if(al.attr('href')){
							var sm = {url:urla(al.attr('href')), name:al.text().trim()};
							data.sections.splice(0, 0 , sm);
						};
						
					});
					
					return JSON.stringify(data);
				}
				
				function section_parseUrl(url, html){  // 把目录里面的每条链接进行补全, 得出一章的多少页
					var $ = cheerio.load(html);
					var urls = [];
					var max = Number($('.manga-page>b').text().slice(1))
					
					for(var i = 1; i < max + 1 ;i++){
						newUrl = url +'?p='+ i ;
						urls.push(newUrl);
					};
					return urls;
				}
				
				function section_parse(url,html) {  // 把每一页的内容解析出来
					var $ = cheerio.load(html);
					var list = [];
					var imgUrl = $('#manga').attr('src');
					list.push(imgUrl)
					return JSON.stringify(list);

				}
				
				
			]]>
		</code>
	</script>
</sited>
```
开发很辛苦, 很多时候都是考脑补来进行Log调试


