---
title: 'java导出PDF,解决linux服务器下中文字体不显示'
date: 2021-04-03 16:24:28
tags: java
categories: java
---

###                java导出PDF,解决linux服务器下中文字体不显示

最近，有个需求导出PDF表格，经过市面上技术的调研，最终选择了itext全家桶。

通过模板freeMark，将HTML渲染到PDF中。支持图片等。

`itextpdf` 作为 pdf 生成核心,`xmlworker` 用于 html 转 pdf,`itext-asian` 用于支持中文显示,视情况而定还可以引入 `nekohtml` 对非标准 html 页面进行标签补全。

1. maven引入核心依赖

```java
  <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itextpdf</artifactId>
            <version>5.5.11</version>
        </dependency>
        <dependency>
            <groupId>com.itextpdf.tool</groupId>
            <artifactId>xmlworker</artifactId>
            <version>5.5.11</version>
        </dependency>
        <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itext-asian</artifactId>
            <version>5.2.0</version>
        </dependency>

```

2、核心代码如下

```java
import com.itextpdf.text.Document;
import com.itextpdf.text.Font;
import com.itextpdf.text.PageSize;
import com.itextpdf.text.pdf.BaseFont;
import com.itextpdf.text.pdf.PdfWriter;
import com.itextpdf.tool.xml.XMLWorkerFontProvider;
import com.itextpdf.tool.xml.XMLWorkerHelper;

import java.io.ByteArrayInputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.charset.Charset;

public class PDFUtil {
    public static void writeStringToOutputStreamAsPDF(String html, OutputStream os) {
        writeToOutputStreamAsPDF(new ByteArrayInputStream(html.getBytes()), os);
    }

    public static void writeToOutputStreamAsPDF(InputStream html, OutputStream os) {
        try {
            Document document = new Document(PageSize.A4);
            PdfWriter pdfWriter = PdfWriter.getInstance(document, os);
            document.open();
            XMLWorkerHelper worker = XMLWorkerHelper.getInstance();
            worker.parseXHtml(pdfWriter, document, html, Charset.forName("UTF-8"), new AsianFontProvider());
            document.close();
        } catch (Exception e) {
        }
    }
}
/**
 * 用于中文显示的Provider
 */
class AsianFontProvider extends XMLWorkerFontProvider {
    @Override
    public Font getFont(final String fontname, String encoding, float size, final int style) {
        try {
            BaseFont bfChinese = BaseFont.createFont("STSongStd-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED);
            return new Font(bfChinese, size, style);
        } catch (Exception e) {
        }
        return super.getFont(fontname, encoding, size, style);
    }
}
```

其中writeStringToOutputStreamAsPDF中Html为通过流读取的字符串，如果项目采用的是jar包形式，放在resource目录下，可能会导致找不到路径。故采用的流方式读取。

AsianFontProvider解决linux下导出中文字体不显示问题。

