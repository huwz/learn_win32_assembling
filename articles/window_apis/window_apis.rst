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

1. 格式化消息文本。
   
函数需要一个定义消息作为输入。
定义消息既可能来自于函数的输入参数。
也可能来自于加载模块的消息表。

2. 搜索系统消息表资源以获取定义消息。
   
函数通过dwMessageId和dwLanguageId查找消息表，获取定义消息；
处理文本中的占位符序列；处理后的文本拷贝到输出缓冲区


**参数**：

*dwFlags*：包含一系列标志位，多方位描述格式化过程，并指明参数 `lpSource` 的解析方法。

dwFlags的低位字节指明输出缓冲区的断行方式和最大格式化行宽度。

有以下几种标志位组合：

* FORMAT_MESSAGE_ALLOCATE_BUFFER
  
  参数lpBuffer是一个PVOID类型的指针，指向输出缓冲区。
  ANSI编码时，nSize指定输出缓冲区的最大字节数；
  UNICODE编码时，nSize指定输出缓冲区的最大字符数。
  
  .. warning:: 需要调用LocalFree()释放lpBuffer。

* FORMAT_MESSAGE_IGNORE_INSERTS
  
  保留消息文本中的占位符，不予处理。
  如果设置了该标识，Arguments参数会被忽略

* FORMAT_MESSAGE_FROM_STRING 
   
  lpSource指向一个 ``'\0'`` 结尾的消息文本。
  文本可以含有占位符。
  
  .. warning:: 不和FORMAT_MESSAGE_FROM_HMODULE或FORMAT_MESSAGE_FROM_SYSTEM一起使用。

* FORMAT_MESSAGE_FROM_HMODULE
   
  表明lpSource是模块句柄，模块中含有消息表资源。
  如果lpSource是NULL，会搜索当前进程的内存映像文件。

  .. warning::
   不和FROMAT_MESSAGE_FROM_STRING一起使用。

* FORMAT_MESSAGE_FROM_SYSTEM
   
  搜索系统消息表获取定义消息。
  如果和FORMAT_MESSAGE_FROM_HMODULE一起使用，则函数先搜索lpSource指定的模块，再到系统消息表中搜索。

  .. warning::
   不和FORMAT_MESSAGE_FROM_STRING一起使用。

* FORMAT_MESSAGE_ARGUMENT_ARRAY
   
  Arguments参数不是va_list结构，而是一个数值(32位)数组。

  dwFlags的低位字节也可以指定格式化输出行的最大宽度。
  使用FORMAT_MESSAGE_MAX_WIDTH_MASK常量和位运算设置和获取该值。

  +-------------------------------------+--------------------------------------------------+
  | dwFlags低位字节                     | 含义                                             |
  +=====================================+==================================================+
  | 0                                   | 输出文本没有宽度限制, 函数保留输出缓冲区的占位符 |
  +-------------------------------------+--------------------------------------------------+
  | 不等于FORMAT_MESSAGE_MAX_WIDTH_MASK | 指明输出行的最大字符数，                         |
  | 的非零值                            | 忽略消息文本中的常规换行符，保留占位符(%n)       |
  +-------------------------------------+--------------------------------------------------+
  | FORMAT_MESSAGE_MAX_WIDTH_MASK       | 忽略定义消息的常规换行符，保留占位符，不产生新行 |
  +-------------------------------------+--------------------------------------------------+


*lpSource*：消息文本指针，类型由dwFlags决定：

 +-----------------------------+--------------------------------+
 | dwFlags                     | lpSource                       |
 +=============================+================================+
 | FORMAT_MESSAGE_FROM_HMODULE | 含有消息表的模块handle         |
 +-----------------------------+--------------------------------+
 | FORMAT_MESSAGE_FROM_STRING  | 未格式化消息文本（LPTSTR类型） |
 |                             | ，函数根据占位符进行格式化     |
 +-----------------------------+--------------------------------+
 | 其他                        | 被忽略                         |
 +-----------------------------+--------------------------------+

*dwMessageId*：32位的消息标识符。

 .. warning:: 当dwFlags包含了标志位FORMAT_MESSAGE_FROM_STRING时，该参数会被忽略。

*dwLanguageId*：32位的语言标识符。

 .. warning:: 当dwFlags包含了标志位FORMAT_MESSAGE_FROM_STRING时，该参数会被忽略。

 根据指定的语言id（LANGID）搜索消息表，成功则返回对应的消息文本；
 失败则返回ERROR_RESOURCE_LANG_NOT_FOUND。

 如果dwLanguageId = 0，则函数依次根据以下语言id先后查找消息表：

  1. LANG_NEUTRAL
  2. 线程默认LANGID，基于用户默认的locale值
  3. 系统默认LANGID，基于系统默认的locale值
  4. 美式英语。

 如果失败，则再根据其他语言id查找消息文本；
 如果还是失败，则返回ERROR_RESOURCE_LANG_NOT_FOUND

*lpBuffer*：指向以 ``\0`` 结束的格式化消息文本。

 如果dwFlags包含FORMAT_MESSAGE_ALLOCATE_BUFFER，则loBuffer指向一个由LocalAlloc()分配输出缓冲区。

*nSize*：

  +----------------------------------------+--------------------------------------------------+
  | 是否设置FORMAT_MESSAGE_ALLOCATE_BUFFER | nSize                                            |
  +========================================+==================================================+
  | 否                                     | 输出缓冲区的最大字节数(ANSI)/最大字符数(UNICODE) |
  +----------------------------------------+--------------------------------------------------+
  | 是                                     | 输出缓冲区的最小字节数(ANSI)/最小字符数(UNICODE) |
  +----------------------------------------+--------------------------------------------------+

*Argument*：数值（32位）数组，用于替换消息文本中的占位符。
 %1表示Argument中的第一个参数，%2表示第二个，以此类推。

 解析该数组取决于和消息文本中插入序列对应的格式化信息，数组元素默认为字符串指针。

 一般情况下Arguments是 `va_list*` 类型。
 如果不用 `va_list*` 类型的指针，可以指定FORMAT_MESSAGE_ARGUMENT_ARRAY标识。
 传递一个32位数值的数组，用于替换消息文本的占位符。

**返回值**：如果函数成功返回，则返回值是输出buffer的字节数(ANSI)或者字符数(UNICODE)，不包括串结束符。

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