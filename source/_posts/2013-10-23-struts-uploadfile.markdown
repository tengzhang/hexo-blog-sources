---
layout: post
title: "struts2文件上传"
date: 2013-10-23 09:02
comment: true
categories: struts
---
注意保持action中File的变量和jsp中input的name名相同。
action中定义变量：

<!-- more -->

``` java
package action;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import com.opensymphony.xwork2.ActionSupport;
public class UploadFileAction extends ActionSupport {
    
    // 上传的文件
    private File upload;
    
    // 上传的文件名
    private String uploadFileName;
    
    // 上传的文件类型
    private String uploadContentType;
    
    public String execute() throws Exception {
        FileInputStream fis = new FileInputStream(upload);
        File file = new File("E:/dsideal/upload/" +  uploadFileName);
        FileOutputStream fos = new FileOutputStream(file);
        byte[] buffer = new byte[8912];
        int count = 0;
        while((count = fis.read(buffer)) != -1) {
            fos.write(buffer, 0, count);
        }
        fos.close();
        fis.close();
        return SUCCESS;
    }
    
    /* set、get method */
}
```
jsp页面:

``` java
<s:form action="mystruts/uploadFile.action" enctype="multipart/form-data">
    <s:file name="upload" label="输入要上传的文件名字" />
    <s:submit value="上传" />
    </s:form>
</body>
```
使用了struts标签，也可以直接使用html表单标签form、input
