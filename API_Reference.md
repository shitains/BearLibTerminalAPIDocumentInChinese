# API参考
详细介绍不同库部分的单独页面（正在进行的工作）：

[配置](http://foo.wyrd.name/en:bearlibterminal:reference:configuration)

[输入](http://foo.wyrd.name/en:bearlibterminal:reference:input)

输出

API由大约20个函数和许多常量组成。

## 初始化和配置

### open
`int terminal_open();`

此函数初始化BearLibTerminal实例，使用默认参数配置窗口：80×25个单元格，Fixedsys Excelsior字体，黑色背景上的白色文本。此功能不会将窗口显示在屏幕上。在第一次调用刷新之前，该窗口不会显示。请注意，除非使用成功的open调用初始化库，否则所有其他库函数都将立即返回，返回代码（如果有）指示错误。

函数返回布尔值，其中false表示初始化失败。详细信息可以在日志文件中找到（默认名称为bearlibterminal.log）。

### close
`void terminal_close();`

与打开相对，此函数关闭窗口并取消初始化BearLibTerminal实例。

### set
`int terminal_set(const char* s); // + setf, wset, and wsetf flavours`

这可能是BearLibTerminal API中最复杂的函数。配置库选项和机制，管理字体、瓷砖集（tileset），甚至配置文件都可以使用它来执行。
该函数接受一个“配置字符串”，其中描述了各种选项：
window.title='game'; //窗体标题
font: UbuntuMono-R.ttf, size=12;  //字体及字体大小
ini.settings.tile-size=16; //初始化瓷砖(tile)设置

有关配置字符串格式、库选项和整体函数行为的信息，请参阅单独的[配置](http://foo.wyrd.name/en:bearlibterminal:reference:configuration)页面。

## 输出状态
### color
`void terminal_color(color_t color);`

此函数设置当前前景颜色，该颜色将由稍后调用的所有输出函数使用。color_t类型是一个32位无符号整数，以BGRA（0xAARRGGBB）格式描述颜色。此数字颜色表示可以直接构造，也可以通过[argb](http://foo.wyrd.name/en:bearlibterminal:reference#color_from_arg)和[颜色名](http://foo.wyrd.name/en:bearlibterminal:reference#color_from_name)函数构造。对于包括C++在内的大多数语言，还有一个重载版本将颜色名称作为字符串（请参见[颜色名函数](http://foo.wyrd.name/en:bearlibterminal:reference#color_from_name)和[特定语言的注释](http://foo.wyrd.name/en:bearlibterminal:language-specific)）。

可以通过读取[TK_COLOR](http://foo.wyrd.name/en:bearlibterminal:reference#state)状态来检索当前前景颜色的数值。

### bkcolor
`void terminal_bkcolor(color_t color);`

与颜色类似，此函数设置当前背景颜色。否则，函数的行为与其前景对应项完全相同。请注意，只有第一即最底层的单元格具有背景。
可以通过读取TK_BKCOLOR状态来检索当前背景色的数值。

### composition
`void terminal_composition(int mode);`

此函数用于设置字符合成模式。当合成关闭时，在单元格中放置瓷砖（通过放置或打印）只需替换该单元格的内容。但是，当“合成”处于启用状态时，该瓷砖将添加到单元格的平铺堆栈中。这具有将多个瓷砖组合成一个瓷砖的视觉效果。对单个单元格中的瓷砖数量没有强制限制。请注意，堆栈中的每个平铺都有自己的前景色和偏移量（请参见put ext）。

参数模式可以是以下模式之一：TK_OFF关闭（默认），TK_ON打开。

可以通过读取TK_COMPOSITION合成状态来检索当前合成模式。

### layer
`void terminal_layer(int layer);`

此函数用于选择角色单元格的当前图层。参数是从0到255的索引，其中0是最低（默认）层。只有第一层有背景，对于第1层及以上，bkcolor设置的背景色没有效果。请注意，“清除区域”（clear area）仅影响当前层，而“清除”（clear）将擦除整个场景。

由于各种原因，图层很有用。一是BearLibTerminal允许比一个字符单元大的平铺。但场景是按固定的从左到右、从上到下的顺序逐单元绘制的。这使得大平铺无法正确覆盖右侧和下方的单元格，因为稍后将绘制这些单元格。使用层，可以将场景分割为几个部分，并在它们之间进行严格的Z排序。

层的另一个用途是从逻辑上分离场景。层可以单独清除和更新，因此可以将场景的一部分放置在单独的层中，并在不接触其他层的情况下进行更新（例如，动画副本级别和静态UI）。

当前层索引可以从TK_LAYER层状态插槽中读取。
