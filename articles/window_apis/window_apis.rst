窗口APIs
========

1. 格式化消息文本
-----------------

.. code-block:: C
  :linenos:

  DWORD FormatMessage(
   DWORD dwFlags
   LPCVOID lpSource
   DWORD dwMessageId,
   LPTSTR dwLanguageId,
   LPTSTR lpBuffer,
   DWORD nSize,
   va_list *Arguments
  )

**功能**：

1. 将指定的消息文本进行格式化。
2. 根据dwMessageId和dwLanguageId搜索系统消息表，获取消息文本。

**参数**：

* dwFlags
  
  高24位包含一系列标志位；
  低8位指明输出缓冲区的断行方式和最大行宽度。

	+--------------------------------+--------------------------------------------------------------------+
	| dwFlags                        | 说明                                                               |
	+================================+====================================================================+
	| FORMAT_MESSAGE_ALLOCATE_BUFFER | 若指定该标识，则分配一个最大字符数不小于nSize的输出缓冲区；        |
	| (0x00000100)                   | 若没有指定，则分配一个最大字符数不大于nSize的输出缓冲区            |
	+--------------------------------+--------------------------------------------------------------------+
	| FORMAT_MESSAGE_IGNORE_INSERTS  | 消息文本不进行格式化，参与格式化数组Arguments被忽略                |
	| (0x00000200)                   |                                                                    |
	+--------------------------------+--------------------------------------------------------------------+
	| FORMAT_MESSAGE_FROM_STRING     | lpSource是一个字符串                                               |
	| (0x00000400)                   |                                                                    |
	+--------------------------------+--------------------------------------------------------------------+
	| FORMAT_MESSAGE_FROM_HMODULE    | lpSource指向某个模块实例，程序会自动搜索该模块中的消息表。         |
	| (0x00000800)                   | 如果lpSource是NULL，则搜索当前进程的内存映像文件。                 |
	+--------------------------------+--------------------------------------------------------------------+
	| FORMAT_MESSAGE_FROM_SYSTEM     | 搜索系统消息表获取定义消息。                                       |
	| (0x00001000)                   |                                                                    |
	+--------------------------------+--------------------------------------------------------------------+
	| FORMAT_MESSAGE_FROM_HMODULE OR | 先搜索lpSource指定的模块的消息表，如果失败则再到系统消息表中搜索。 |
	| FORMAT_MESSAGE_FROM_HMODULE    |                                                                    |
	+--------------------------------+--------------------------------------------------------------------+
	| FORMAT_MESSAGE_ARGUMENT_ARRAY  | Arguments是32位的数值数组                                          |
	+--------------------------------+--------------------------------------------------------------------+

	.. warning:: FORMAT_MESSAGE_FROM_STRING 和 FORMAT_MESSAGE_FROM_HMODULE 或者 FORMAT_MESSAGE_FROM_SYSTEM
	   不能混合使用

	+-------------------------------------+----------------------------------+
	| dwFlags低8位                        | 输出文本断行规则                 |
	+=====================================+==================================+
	| 0                                   | 不断行；                         |
	|                                     | 保留常规断行符                   |
	+-------------------------------------+----------------------------------+
	| 不等于FORMAT_MESSAGE_MAX_WIDTH_MASK | 一行中字符数达到该值时自动换行； |
	| 的非零值                            | 将行中常规断行符用%n转义序列编码 |
	+-------------------------------------+----------------------------------+
	| FORMAT_MESSAGE_MAX_WIDTH_MASK       | 不断行；                         |
	|                                     | 将行中常规断行符用%n转义序列编码 |
	+-------------------------------------+----------------------------------+

* dwMessageId/dwLanguageId
  
  32位的消息标识符/语言标识符。

  .. warning:: 当dwFlags包含了标志位FORMAT_MESSAGE_FROM_STRING时，该参数会被忽略。

  根据指定的语言id（LANGID）搜索消息表，成功则返回对应的消息文本；
  失败则返回ERROR_RESOURCE_LANG_NOT_FOUND。

  如果dwLanguageId = 0，则函数会按照先后次序使用以下id查找消息表：

  1. LANG_NEUTRAL
  2. 线程默认LANGID，基于用户默认的locale值
  3. 系统默认LANGID，基于系统默认的locale值
  4. 美式英语。
  5. 其他语言id

  如果失败，则返回ERROR_RESOURCE_LANG_NOT_FOUND

* lpBuffer
  
  存储输出文本。

* Argument 
  
  用于替换消息文本中格式化序列的数组。
  %1对应Argument中的第一个元素，%2对应第二个元素，...

  一般情况下Arguments是 `va_list*` 类型。
  如果使用FORMAT_MESSAGE_ARGUMENT_ARRAY标识，则Arguments必须为32位数值数组

**返回值**：
  
  如果函数成功返回，则返回值是消息文本字符数，不包括结束符 ``\0``。

**注意**：
 
 打印系统错误信息的方法：

.. code-block:: C
  :linenos:

  LPVOID lgMsgBuf;
  FormatMessage(
  FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM,
  NULL,
  GetLastError(),
  MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT)
  (LPTSTR)&lpMsgBuf,
  0,
  NULL
  );