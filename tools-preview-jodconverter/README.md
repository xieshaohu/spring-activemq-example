## 简述

在Web项目中，我们通常需要在线预览文档，但是上传的文档附件有很多种格式，浏览器不一定能够支持预览，此时可以借助此示例工程进行附件格式转换。

此工程基本原理是获取到原始文件流之后，调用libreoffice进行格式转换，输出成设定的目标格式，目标格式可以是html、PDF等浏览器能够直接进行预览的的格式。

如果设定的是html，原始文档中的图片会以base64的方式直接嵌入html源文件中，为了增加预览的兼容性，建议设置成PDF。

工程中如果碰到不能支持的格式，绕过此接口，直接在浏览器下载原始文件。例如rar，zip等不能预览的格式。

示例工程会启动4个libreoffice进程，所以是支持并发进行附件转换和预览的，官方给出的转换性能如下：

| Document                                                     | Size   | Pages | Avg Time (ms) | Throughput (per minute) |
| ------------------------------------------------------------ | ------ | ----- | ------------- | ----------------------- |
| Hello World!                                                 | 7 kb   | 1 p   | 98 ms         | 612 p/m                 |
| [Metadata Use Cases and Requirements](http://www.oasis-open.org/committees/download.php/20492/UCR.odt) | 13 kb  | 5 p   | 710 ms        | 422 p/m                 |
| [Open Document Format v1.1 Accessibility Guidelines](http://docs.oasis-open.org/office/office-accessibility/v1.0/cd01/ODF_Accessibility_Guidelines-v1.0.odt) | 81 kb  | 52 p  | 2314 ms       | 1348 p/m                |
| [OpenDocument v1.1 Specification](http://docs.oasis-open.org/office/v1.1/OS/OpenDocument-v1.1.odt) | 475 kb | 737 p | 24084 ms      | 1836 p/m                |

> Tests made with jodconverter-cli and Apache OpenOffice 4.1.3 on a laptop with a quad-core Intel(R) Core(TM) i7-6500U CPU @ 2.50GHz processor and Windows 10

## 必要条件

- 服务器上安装libreoffice，使用命令`yum -y install libreoffce`安装，不推荐OpenOffice，因为已经停止维护了
- 默认JDK1.8，JDK11也测试通过了

## 支持的格式转换
| 文档类型 | 输入格式                         | 输出格式                                                |
| ------------- | ------------------------------------ | ------------------------------------------------------------ |
| Text          | DOC, DOCX, ODT, OTT, RTF, TEXT, etc. | DOC, DOCX, HTML, JPG, ODT, OTT, FODT, PDF, PNG, RTF, TXT, etc. |
| Spreadsheet   | CSV, ODS, OTS, TSV, XLS, XLSX, etc.  | CSV, HTML, JPG, ODS, OTS, FODS, PDF, PNG, TSV, XLS, XLSX, etc. |
| Presentation  | ODP, OTP, PPT, PPTX, etc.            | GIF, HTML, JPG, ODP, OTP, FODP, PDF, PNG, PPT, PPTX, BMP, etc. |
| Drawing       | ODG, OTG, etc.                       | GIF, JPG, ODG, OTG, FODG, PDF, PNG, SVG, TIF, VSD, BMP, etc. |
| Other         | HTML                                 | DOC, DOCX, HTML, JPG, ODT, OTT, FODT, PDF, PNG, RTF, TXT, etc. |

## 启动方法

maven package之后，上传可执行的jar包放到安装好了libreoffice的服务器，然后执行下面的命令。

```shell
java -jar preview-jodconverter-4.3.0.RELEASE.jar
```

## 转换文档的两种方法

1. 请求`/lool/convert-to/pdf`在URL的最后一段指定格式，返回的response内容就是指定格式的内容

2. 请求`/lool/convert-to，在参数`format`中指定文件格式，返回的reponse内容也是指定格式的内容

   以上两种方法都需要传递`name`为`data`的文件流，以下给出html样例

   第1种方法样例：

   ```html
   <html>
       <body>
           <form action="http://0.0.0.0:10000/lool/convert-to/pdf" enctype="multipart/form-data" method="post">
             File: <input type="file" name="data"><br/>
             <input type="submit" value="Convert">
           </form>
       </body>
   </html>
   ```
   
   第2种方法样例：

   ```html
   <html>
       <body>
           <form action="http://0.0.0.0:10000/lool/convert-to" enctype="multipart/form-data" method="post">
             File: <input type="file" name="data"><br/>
             Format: <input type="text" name="format"><br/>
             <input type="submit" value="Convert">
           </form>
       </body>
   </html>
   ```

**重要说明：**可以通过`application.yml`文件中的`browser.type`控制浏览器是下载转换后的文档还是直接在浏览器中打开转换后的文档；`browser.type`作用与HTTP Response Header的`Content-Disposition`参数`inline`是在浏览器中打开，`attachment`是浏览器提示下载

## 参考资料

- 此工程参考github开源项目 [jodconverter-sample-rest](https://github.com/sbraconnier/jodconverter/tree/master/jodconverter-samples/jodconverter-sample-rest) 的4.3.0分支版本