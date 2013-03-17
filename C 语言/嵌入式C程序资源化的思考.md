嵌入式C程序资源化的思考
=======================

这里不考虑那些可以使用库的系统，比如可以使用 gettext 的 linux 系统，这里讨论的是一些
很简单的系统，比如 ucos，以及很简单的界面库 ucgui。

一般的做法是将需要翻译的字符串汇集到一个数组中，然后使用函数来引用字符串，为了增加可
读性，还会使用枚举。这里有篇文章讲解这种做法：[Use Strings to Internationalize C Programs][1].

这种方法比较直接，优点是占用资源不高，很符合嵌入式的环境。缺点也明显：

*	手动量比较大。当然如果字符串少的话也没什么问题。添加一个新的字符串要在几处做更改，
	还需要保证数组的顺序。
*	不适合自动化生成。自动化并不能给你生成带有意义的枚举名，而且自动化脚本需要修改源文件，
	这会引发其他问题。
*	不直观，当维护时，源代码中的字符串看起来就没那么直观了，尤其是接手维护此代码的开发者。

因为用过 [Qt][2] ，对于它那种资源化/国际化方式特别喜欢，直观且优雅。后来了解了
下 [GNU gettext][3] ，大体应该与 Qt 使用的方法类似。

自己也试着思考了下这些工具内部实现的原理。以 Qt 的 `tr()` 函数举例：

    QString str = tr("C语言");

`tr()` 函数在运行时取出显示用的字符串，它使用的参数就是源代码中使用的字符串本身，这里就
是`"C语言"`，`tr()` 也应该是从一张表中查找翻译语言对应的字符串，没找到就返回它的参数。
对于这个表查找应该使用了[字符串hash][4] 以提高查找效率。对于表的生成使用了外部工具对源代码
进行处理，也就是自动化生成。

下面讲下我准备的思路：

1.	源代码中需要翻译的字符串用一个宏封装，比如 `_TR("some stting")` 
	
	    #define _TR(str) tr(str)

	`tr()` 就是查找函数，用宏的好处在于灵活性。
2.	字符串提取可以写个脚本，遍历所有的 C文件，逐行处理提取出所有的字符串，也可以顺带一些
	位置信息，比如提取自哪个文件哪一行。
3.	`tr()` 函数查找的表是一个只读数组，数组元素类型定义大概如下：

	    enum Language {
	        LANG_DEFAULT,
 	        LANG_ZH_CN,
 	        LANG_EN,
	        // ...
	        LANG_LAST
 	    };
	        
	    struct UniversalString  {
	        int hash; 
 	        const char * const str[LANG_LAST];
		};


	结构定义 `struct UniversalString` 就是数组元素类型了，`hash` 的作用是为了加速字符串查找，
	使用良好的字符串hash函数的话，碰撞率很低，一个 `hash` 域就足够了，否则可以添加多个存储
	使用不同 hash 函数产生的值。
4.	表数组由脚本自动生成，自动产生字符串 hash 值，用原始字符串填充 `str[LANG_DEFAULT]`，这样
	设计是因为字符串 hash 算法并不是 1对1，可以再进行一次字符串比较，这比直接在整个表中遍历
	逐个比较效率要高，而且还有个好处，对于翻译人员，有原始字符串更方便（这也可以通过添加注释
	实现）。如果想避免字符串比较，可以存储2个或多个不同 hash 函数的值，若这些值都相同，就认定
	字符串相同，这是个“概率保证”，如果 hash 函数足够好，这个认定失败的概率很低。一个示范表定义

	    static const struct UniversalString strings[] = {
	    	{// ...\src\a.c:114 
	    		15,
	    		{ 
	    			"C语言",
	                "",
	    			//...
	    		}
	    	},
	    	{// ...\src\a.c:145
	    		1563,
	    		{
	    			"some string",
	    			"",
	    			// ...
	    		}
	    	},
	    	// ...
	    };
	
5.	函数 `tr()` 代码大概如下，其中 `find()` 就是查找函数，`get_lang()` 返回的是当前的语言，返回值
	是枚举 `Language` 中的取值。

	    const char * tr(const char * str)
	    {
	    	const struct UniversalString * ps = find(str);
	    	if(ps) { // found
	    		return ps.str[get_lang()];
	    	} else { // not exist
	    		return str;
	    	}
	    }
	
	`find()` 函数就不写出了，这个函数优化的好，整体效率就不错。

6.	如果是修改代码时候修改了字符串，重新生成时自动化工具应该可以保留一些未改动的字符串的已翻译
	内容，这是写工具要注意的事情。

------------

参考资料:

1. [Use Strings to Internationalize C Programs][1].
2. [使用GetText本地化编程][5].	 
3. [各种字符串Hash函数比较][4].

[1]: http://www.rmbconsulting.us/use-strings-to-internationalize-c-programs
[2]: http://qt-project.org
[3]: http://www.gnu.org/software/gettext/
[4]: http://www.byvoid.com/blog/string-hash-compare/
[5]: http://jianlee.ylinux.org/Computer/C/gettext.html

