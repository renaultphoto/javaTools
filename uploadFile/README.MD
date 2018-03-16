### 1、写在前面
文件上传是大部分系统必备的功能，如OA系统中财务报销需要上传报销单据、技术论坛需要上传安装包附件、知识管理系统需要上传知识文档等。
现在有一个商城的项目，项目中商品的图片都是通过商家自己上传、用户头像也是用户自己维护的，但是项目中图片均是通过七牛云API上传到云空间上的，因为涉及一些用户的个人信息，特此需要存在本地服务器上，然后记录一下怎么写的通用上传组件。
### 2、实现目标
![页面](https://upload-images.jianshu.io/upload_images/3580580-103acd54f1e5548e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![上传成功](https://upload-images.jianshu.io/upload_images/3580580-cf8df5be703b9fe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、页面代码

```
<!DOCTYPE html>
<html>
<head>
<meta
 http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<title>上传文件demo</title>
<script type="text/javascript" src="<%=path%>/js/comm/jquery.min.js"></script>
<script type="text/javascript" src="<%=path%>/js/comm/ajaxfileupload.js"></script>
 <style>
.upLoadCss{   
    position: relative;
    display: inline-block;
    background: #D0EEFF;
    border: 1px solid #99D3F5;
    border-radius: 4px;
    padding: 4px 12px;
    overflow: hidden;
    color: #1E88C7;
    text-decoration: none;
    text-indent: 0;
    line-height: 20px;}
    .upLoadCss input {
    position: absolute;
    font-size: 100px;
    right: 0;
    top: 0;
    opacity: 0;}
    .upLoadCss:hover {
    background: #AADFFD;
    border-color: #78C3F3;
    color: #004974;
    text-decoration: none;}
</style>
<script>
//用于进行图片上传，返回地址 
function setImg(obj){
	$("#uploadInfo").html("LOGO正在上传...");
    var f=$(obj).val();
    if(f == null || f ==undefined || f == ''){
        return false;
    }
  //如果只是上传图片可以用此方法校验
 /*    if(!/\.(?:png|jpg|bmp|gif|PNG|JPG|BMP|GIF)$/.test(f))
    {
        alertLayel("类型必须是图片(.png|jpg|bmp|gif|PNG|JPG|BMP|GIF)");
        $(obj).val('');
        return false;
    } */
    var data = new FormData();
    $.each($(obj)[0].files,function(i,file){
        data.append('file', file);
    });
    $.ajax({
        type: "POST",
        url: "/sup/uploadFile.do",
        data: data,
        cache: false,
        contentType: false,    //不可缺
        processData: false,    //不可缺
        dataType:"json",
        success: function(suc) {
        	console.log(suc);
        	alert(suc);
        	$("#uploadInfo").html("LOGO上传成功");
        	$("#thumburl").val(suc);//将地址存储好
        	//$("#uploadInfo").append(suc);//显示上传成功的文件名
          //  $("#thumburlShow").attr("src",suc);//如果显示图片，不用注释
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
            alert("上传失败，请检查网络后重试");
            $("#url").val("");
            $(obj).val('');
        }
    });
}
</script>
</head>
<body>
<table>
	<tr>
		<td>图片</td>
		<td>
			<input type="hidden" id="thumburl" name=""  value=""/>
			<img  id="thumburlShow" style="width:100px;height:80px" name="" src="/img/default_goods_image_240.gif">
			<a href="javascript:;" class="upLoadCss">更换LOGO
				<input type="file" name="logoFile" id="logoFile" onchange="setImg(this);">
			</a>
			<span id="uploadInfo"></span>
		</td>
	</tr>
</table>
</body>    
</html>
```

### 4、后台代码
```
 /**
		   * 上传文件到指定目录并返回url
		   * @param file
		   * @return
		   * @throws IOException
		   */
		  @RequestMapping(value = "/uploadFile",produces="application/json;charset=UTF-8",method = RequestMethod.POST)
		  @ResponseBody
		  public String uploadFile(@RequestParam(value="file",required=false)MultipartFile file) throws IOException {
		        //本地使用,上传位置
		        String rootPath ="D:/files/";
		        //文件的完整名称,如spring.jpeg
		        String filename = file.getOriginalFilename();
		        //文件名,如spring
		        String name = filename.substring(0,filename.indexOf("."));
		        //文件后缀,如.jpeg
		        String suffix = filename.substring(filename.lastIndexOf("."));

		        //创建年月文件夹
		        Calendar date = Calendar.getInstance();
		        File dateDirs = new File(date.get(Calendar.YEAR)
		                + File.separator + (date.get(Calendar.MONTH)+1));
		        
		        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddhhmmssSSS");
		    	String filedate = sdf.format(new Date());
		    	filename = name+filedate+suffix;
		        //目标文件
		        File descFile = new File(rootPath+File.separator+dateDirs+File.separator+filename);
		        int i = 1;
		        //若文件存在重命名
		        String newFilename = filename;
		        while(descFile.exists()) {
		            newFilename = name+"("+i+")"+suffix;
		            String parentPath = descFile.getParent();
		            descFile = new File(parentPath+File.separator+newFilename);
		            i++;
		        }
		        //判断目标文件所在的目录是否存在
		        if(!descFile.getParentFile().exists()) {
		            //如果目标文件所在的目录不存在，则创建父目录
		            descFile.getParentFile().mkdirs();
		        }
		        //将内存中的数据写入磁盘
		        file.transferTo(descFile);
		        //完整的url
		        String fileUrl =  "/files/"+dateDirs+ "/"+newFilename;

		        return fileUrl;
		    }
```

### 后续

[项目源码： java实现文件上传]()
[ Tomcat配置服务器目录映射到本地磁盘目录]()
[java实现七牛云API上传云空间]()



