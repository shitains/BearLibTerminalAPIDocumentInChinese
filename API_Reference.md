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
可以通过读取[TK_BKCOLOR状态](http://foo.wyrd.name/en:bearlibterminal:reference#state)来检索当前背景色的数值。

### composition
`void terminal_composition(int mode);`

此函数用于设置字符合成模式。当合成关闭时，在单元格中放置瓷砖（通过[put](http://foo.wyrd.name/en:bearlibterminal:reference#put)或[print](http://foo.wyrd.name/en:bearlibterminal:reference#print)）只需替换该单元格的内容。但是，当“合成”处于启用状态时，该瓷砖将添加到单元格的平铺堆栈中。这会把多个瓷砖组合成一个瓷砖。单个单元格中的瓷砖数量没有限制。请注意，堆栈中的每个瓷砖都有自己的前景色和偏移量（请参见[put ext](http://foo.wyrd.name/en:bearlibterminal:reference#put_ext)）。

参数模式可以是以下模式之一：TK_OFF（默认），TK_ON。

可以通过读取[TK_COMPOSITION状态](http://foo.wyrd.name/en:bearlibterminal:reference#state)来检索当前合成模式。

### layer
`void terminal_layer(int layer);`

此函数用于选择角色单元格的当前图层。参数是一个取值从0到255的索引，其中0是最低（默认）层。只有第一层有背景，对于第1层及以上，bkcolor设置的背景色没有效果。请注意，[clear_area](http://foo.wyrd.name/en:bearlibterminal:reference#clear_area)仅影响当前层，而[clear](http://foo.wyrd.name/en:bearlibterminal:reference#clear)将清除所有层。

由于各种原因，图层很有用。一是BearLibTerminal允许比一个字符单元大的瓷砖。但场景是按固定的从左到右、从上到下的顺序逐单元绘制的。这使得大瓷砖无法正确覆盖右侧和下方的单元格，因为稍后将绘制这些单元格。使用层，可以将场景分割为几个部分，并在它们之间进行严格的Z排序。

层的另一个用途是从逻辑上分离场景。层可以单独清除和更新，因此可以将场景的一部分放置在单独的层中，并在不接触其他层的情况下进行更新（例如，动画副本级别和静态UI）。

当前层索引可以从TK_LAYER状态中读取。

## 输出
### clear
`void terminal_clear();`

此函数用于清除整个场景（所有[层](http://foo.wyrd.name/en:bearlibterminal:reference#layer)）。它还将每个单元格的背景色设置为当前选定的背景色。

### clear_area
`void terminal_clear_area(int x, int y, int w, int h);`

此函数用于清除当前选定[层](http://foo.wyrd.name/en:bearlibterminal:reference#layer)的一部分。参数指定左上角和要清除的矩形区域的大小。在第一层上调用时，它还会将受影响单元格的背景色设置为当前选定的背景色。

### crop
`void terminal_crop(int x, int y, int w, int h);`

此函数用于设置当前[层](http://foo.wyrd.name/en:bearlibterminal:reference#layer)的裁剪区域。该区域的尺寸以单元格表示。通过将区域的宽度或高度设置为零或使用“clear()”清除整个场景，可以禁用裁剪。

### refresh
`void terminal_refresh();`

此函数用于提交场景以进行输出。它还具有重绘场景的效果。
BearLibTerminal不会在[put]()或[print]()调用时立即绘制到屏幕。相反，场景是以双缓冲方式在屏幕外构建的。如果窗口的内容由于某种原因被破坏（例如，窗口被阻塞，操作系统要求其刷新），BearLibTerminal会重新绘制已提交的“前缓冲区”场景。只有在调用此刷新功能时，修改后的场景才会真正显示在屏幕上。
自库初始化后首次调用此函数将在屏幕上显示窗口。在打开和第一次刷新调用之间，窗口保持不可见。

### put
`void terminal_put(int x, int y, int code);`

此函数将与给定字符值关联的瓷砖放在坐标为x,y的单元格中。如果指定的代码与任何瓷砖都没有关联，则库使用带有细矩形的非字符瓷砖。
请注意，字符代码必须是Unicode值。即使当前选定的字体是仅包含少量瓷砖的位图字体，BearLibTerminal也会在内部将这些瓷砖映射到适当的Unicode值。例如，除非库配置错误，否则在值U+20AC处始终可以使用欧元符号（如果存在）。

### pick
`int terminal_pick(int x, int y, int index);`

此函数用于返回当前图层上指定单元格中符号/瓷砖的代码。index参数指定单元格中瓷砖的索引。如果单元格中没有此类瓷砖（或单元格为空），则函数将返回0。因此，要枚举单元格中的瓷砖，应该递增索引，直到返回零为止。
该函数根据终端对瓷砖代码进行反向代码页翻译。编码选项。
