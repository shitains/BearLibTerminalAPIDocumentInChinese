#API参考
详细介绍不同库部分的单独页面（正在进行的工作）：
[配置](http://foo.wyrd.name/en:bearlibterminal:reference:configuration)
[输入](http://foo.wyrd.name/en:bearlibterminal:reference:input)
输出

API由大约20个函数和许多常量组成。

##初始化和配置

###open
'int terminal_open();'
此函数初始化BearLibTerminal实例，使用默认参数配置窗口：80×25个单元格，Fixedsys Excelsior字体，黑色背景上的白色文本。此功能不会将窗口显示在屏幕上。在第一次调用刷新之前，该窗口不会显示。请注意，除非使用成功的open调用初始化库，否则所有其他库函数都将立即返回，返回代码（如果有）指示错误。

函数返回布尔值，其中false表示初始化失败。详细信息可以在日志文件中找到（默认名称为bearlibterminal.log）。

###close
'void terminal_close();'
与打开相对，此函数关闭窗口并取消初始化BearLibTerminal实例。

###set
'int terminal_set(const char* s); // + setf, wset, and wsetf flavours'
这可能是BearLibTerminal API中最复杂的函数。配置库选项和机制，管理字体、瓷砖集（tileset），甚至配置文件都可以使用它来执行。
该函数接受一个“配置字符串”，其中描述了各种选项：
window.title='game'; //窗体标题
font: UbuntuMono-R.ttf, size=12;  //字体及字体大小
ini.settings.tile-size=16; //初始化瓷砖(tile)设置
有关配置字符串格式、库选项和整体函数行为的信息，请参阅单独的[配置](http://foo.wyrd.name/en:bearlibterminal:reference:configuration)页面。
