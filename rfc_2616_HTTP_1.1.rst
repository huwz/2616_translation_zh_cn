rfc 2616 直译
=============

3.4 字符集
----------

HTTP 对术语“字符集”的定义和 MIME 中描述的一样::
 
 这篇文档中使用的术语“字符集”是指转换方法，通过一张或者多张表格将字节序列转化成字符序列。
 注意，无条件的反向转换不是必须的，在这个过程中，不是所有字符都在指定字符集中，字符集可能提供了多个字节序列表示某个特殊字符。
 该定义旨在允许多种字符编码，从简单的单表映射，比如 ``US-ASCII``，到复杂的表格切换方法，比如那些使用 ``ISO-2022`` 技术的方法。
 然而，和 MIME 字符集名称关联的定义完全确立了从字节到字符的映射方法。
 特别的，使用外部轮廓描述信息决定准确映射行为是不允许的。

.. note::
 术语“字符集”更多的用来表示“字符编码”。
 然而，由于 HTTP 和 MIME 共享同一个注册表，因而术语也是共享的。

HTTP 字符集由不区分大小写的符号标识。
完整的符号集合由 IANA 字符集注册表定义。

``charset = token``

尽管 HTTP 允许用任意的符号表示字符集，但凡使用 IANA 字符集注册表中的项作前缀的符号，
必须表示表中定义的字符集。
实际应用对字符集的使用必须受限于 IANA 注册表中的定义。

3.4.1 缺失字符集
----------------

某些 HTTP/1.0 软件将不带字符集参数的 ``Content-Type`` 头错误理解成“接收方必须靠猜”。
发送者希望避免这种行为，即使字符集是 ``ios-8859-1``，也包含一个字符集参数。
即使不会对接收方造成困扰，也要这么做。

不幸的是，一些老的 HTTP/1.0 客户端处理不会显式指定字符集参数。
HTTP/1.1 接收者必须遵循发送者指定的字符集标签；
开始显示文档时，这些准备猜测的用户代理必须使用 ``content-type`` 字段中的字符集，只要他们支持该字符集。
不能使用接收方的偏好设置。

3.5 内容编码
------------

内容编码的值表示已经应用到或者可以应用到实体的编码转换。
内容编码的主要用途是对文档进行压缩或者在不丢失实体本来的媒体类型和信息的前提下，对文档进行成功转换。
通常，实体以码字形式存储，可以直接发送，只是在接收方进行解码。

``content-coding = token``

所有的 ``content-coding`` 值都是不区分大小写的。
HTTP/1.1 在 ``Accept-Encoding`` 头和 ``Content-Encoding`` 字段中使用 ``content-coding`` 值。
尽管它的值描述了内容编码，但是更为重要的是，它指明了解除编码需要采用什么解码机制。

英特网数字分配机构（IANA）将内容编码值标识符进行了注册。
原始的注册表中只含有以下标识：

* gzip
  
  一种编码格式，由文件压缩程序 ``gzip`` (GUN zip)产生，在 RFC 1952 中有具体描述。
  这种格式是一种 ``Lempel-Ziv`` 编码(LZ77)，使用 32 位的 CRC 校验（冗余循环校验码）。

* compress
  
  这种编码方式由公共 UNIX 文件压缩程序 ``compress`` 产生。
  这种格式是适应式 ``Lempel-Ziv-Welch`` 编码（LZW）。

  使用程序名标识编码格式是不尽人意的，在未来的编码中不鼓励使用这种方式。
  这里作为历史实践的代表使用，不是好的设计。
  出于对 HTTP 老版本的兼容性，实际应用应该考虑用 ``x-gzip`` 和 ``x-compress`` 代替 ``gzip`` 和 ``compress``。

* deflate
  
  RFC 1950 中定义的 ``zlib`` 格式联合 RFC 1951 中定义的 ``deflate`` 压缩。

* identity
  
  默认的（实体）编码方式；在不使用任何转化方式。
  这种内容编码方式只用在 ``Accept-Encoding`` 头中，不能用在 ``Content-Encoding`` 中。

新的 ``content-encoding`` 标识必须进行注册；允许客户端和服务器之间的交互。
需要执行新值的编码算法说明应该是公开的，算法可以独立执行，而且明确其目的是本节中定义的内容编码。

14 头字段定义
-------------

该章节定义了所有标准 HTTP/1.1 头字段的语法和语义。
根据实体头字段的内容，发送和接收是指客户端还是服务器，取决于实体是谁发的，又是谁收的。

14.1 Accept
-----------

``Accept`` 请求头指定响应内容可以接受的某些媒体类型。
``Accept`` 头表示用户请求受限于一小块媒体类型的集合；
比如请求一幅内嵌的图片。

.. code-block:: text

    Accept                       = "Accept" ":"
                                   # (media-range [accept-params] )
    media-range                  = ( "*/*"
                                   | ( type "/" "*" )
                                   | ( type "/" subtype )
                                   ) *( ";" parameter )
    accept-params                = ";" "q" "=" qvalue *( accept-extension )
    accept-extension             = ";" token [ "=" ( token | quoted-string ) ]

``*`` 将媒体类型分组为若干范围， ``*/*`` 表示所有媒体类型， ``type/*`` 表示某个类型的所有子类。
``media-range`` 中可以有多个媒体类型参数，都应用到该范围上。

每个 ``media-range`` 后面可以跟一个或者多个 ``accept-params``，以 ``q`` 参数开始，表示相对质量因子。
第一个参数 ``q`` （有的话）将 ``media-range`` 参数和 ``accept-params`` 参数分开。
质量因子允许用户或者用户代理针对 ``media-range`` 设定一个相对偏好程度，q 的取值范围是 0~1。
默认值为 q=1。

.. note:: 注意：

 用参数名 ``q`` 将媒体类型参数和 ``Accept`` 扩展参数分开的做法归结于历史实践。
 尽管这样做可以防止某个媒体范围的型参数取名为 ``q``，
 但是在 IANA 媒体类型注册中缺少 ``q`` 参数和 ``Accept`` 中几乎不用媒体类型参数是不可能的。
 将来的媒体类型在注册任何参数时，不鼓励使用名称 ``q``。

例子：

``Accept: audio/*; q=0.2, audio/basic``

应该被解析成 “我更希望得到 ``audio/basic`` 类型，但在质量下调 80% 之后，如果有匹配的视频类型，也发给我”。

如果客户端没有提供 ``Accept`` 头，则表示客户端接受所有媒体类型。
如果客户端提供了 ``Accept`` 头，而服务器找不到一个合适的响应，则应该返回 406 号响应（不接受）。

一个更复杂的例子：

.. code-block:: text
 
 Accept: text/plain; q=0.5, text/html,
          text/x-dvi; q=0.8, text/x-c

口头上解析为“客户端更希望得到 ``text/html`` 和 ``text/x-c`` 媒体类型，没有的话，可以发送 ``text/x-dvi`` 实体；
仍旧没有的话，可以发送 ``text/plain`` 实体”。

媒体范围可以用更具体的媒体范围或者媒体类型改写。
如果有多个媒体范围应用于同一个媒体类型，则最具体的参考范围具有更高优先级。

.. note:: 参数越多越具体

例如：

``Accept: text/*, text/html, text/html; level = 1, */*``

有如下的优先级：

1. ``text/html; level=1``
2. ``text/html``
3. ``text/*``
4. ``*/*``

给定一个类型，通过查找与该类型匹配且优先级最高的媒体范围，可以决定类型的质量因子。
例如：

.. code-block:: text

 Accept: text/*;q=0.3, text/html;q=0.7,text/html;level=1,
          text/html;level=2;q=0.4,*/*;q=0.5

会得到如下关联的质量因子：

.. code-block:: text

    text/html;level=1               = 1
    text/html                       = 0.7
    text/plain                      = 0.3
    image/jpeg                      = 0.5
    text/html;level=2               = 0.4
    text/html;level=3               = 0.7

注意：对于特定媒体范围，用户代理可能会得到一系列默认的质量值。
然而，除非用户代理是一个封闭系统，不和其他渲染代理交互，否则，这些是可以由用户手动配置的。

14.2 Accept-Charset
-------------------

``Accept-Charset`` 请求头表示响应体接受的字符集。
该字段允许客户端发信号给服务器，表明它在解析更全面或者目的更具体的字符集方面的能力。
服务器可以使用该字符集将文档展现出来。

.. code-block:: text

    Accept-Charset = "Accept-Charset" ":"
                     1#( ( charset | "*" )[ ";" "q" "=" qvalue ] )``

字符集的值在 3.4 节中有描述。
每个字符集可以跟一个相关的质量值，表征了用户对该字符集的偏好程度。
默认值为 q=1。
例子：

``Accept-Charset: ios-8859-5, unicode-1-1;q=0.8``

如果 ``*`` 出现在 ``Accept-Charset`` 字段中，表示匹配每一种字符集（包括 ``ios-8859-5``)。
这些字符集只在 ``Accept-Charset`` 中提及。
如果 ``Accept-Charset`` 中没有 ``*``，则所有不被显式提及的字符集的质量值为 0，但 ``ios-8859-5`` 除外。
后者若没有显式提到，其质量值为 1。

如果头字段中没有 ``Accept-Charset``，则默认为所有的字符集都是可接受的。
如果 ``Accept-Charset`` 头存在，而服务器找不到可以接受的响应，虽然发送一个不能接受的响应也是允许的，但服务器还是应该返回 406 号响应（不接受），

14.3 Accept-Encoding
--------------------

``Accept-Encoding`` 请求头和 ``Accept`` 头相似，但是限制 ``content-codings`` （3.5节）为响应接受的编码。

.. code-block:: text

    Accept-Encoding = "Accept-Encoding" ":"
                    1#( codings [ ";" "q" "=" qvalue ] )
    codings         = ( content-coding | "*" )``

用法举例：

.. code-block:: text

    Accept-Encoding: compress, gzip
    Accept-Encoding:
    Accept-Encoding: *
    Accept-Encoding: compress;q=0.5, gzip;q=1.0
    Accept-Encoding: gzip;q=1.0, identity; q=0.5, *;q=0



