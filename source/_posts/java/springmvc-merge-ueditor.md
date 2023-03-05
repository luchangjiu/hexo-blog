---
title: springmvc整合百度ueditor富文本编辑器
date: 2020-06-18 20:27:17
tags: java
---

# 说明
springmvc 页面整合百度ueditor 富文本编辑器， 使用spring自带的上传组件，替换掉ueditor 自带的jsp 上传组件， 下载[ueditor](http://ueditor.baidu.com/website/download.html)， 下载jsp 版本utf-8 格式的好了

## 步骤一 搭建项目
这里我为了方便使用了springboot 项目，在`resource/static/`文件夹下创建文件夹ueditor，
然后解压下载好的ueditor1_4_3_3-utf8-jsp.zip文件， 将文件内容复制到ueditor 目录下，
如图：  
![alt 文件](/images/archives/springmvc-merge-ueditor-img01.png)


## 步骤二 编辑上传配置以及代码
修改配置文件 ueditor.config.js
```js
(function () {

    /**
     * 编辑器资源文件根路径。它所表示的含义是：以编辑器实例化页面为当前路径，指向编辑器资源文件（即dialog等文件夹）的路径。
     * 鉴于很多同学在使用编辑器的时候出现的种种路径问题，此处强烈建议大家使用"相对于网站根目录的相对路径"进行配置。
     * "相对于网站根目录的相对路径"也就是以斜杠开头的形如"/myProject/ueditor/"这样的路径。
     * 如果站点中有多个不在同一层级的页面需要实例化编辑器，且引用了同一UEditor的时候，此处的URL可能不适用于每个页面的编辑器。
     * 因此，UEditor提供了针对不同页面的编辑器可单独配置的根路径，具体来说，在需要实例化编辑器的页面最顶部写上如下代码即可。当然，需要令此处的URL等于对应的配置。
     * window.UEDITOR_HOME_URL = "/xxxx/xxxx/";
     */
    var URL = window.UEDITOR_HOME_URL || getUEBasePath();

    /**
     * 配置项主体。注意，此处所有涉及到路径的配置别遗漏URL变量。
     */
    window.UEDITOR_CONFIG = {

        //为编辑器实例添加一个路径，这个不能被注释
        UEDITOR_HOME_URL: URL

        // 服务器统一请求接口路径
        // , serverUrl: URL + "jsp/controller.jsp"  将这里改成下面的代码
        , serverUrl: URL + "/ueditor"
// ...... 省略下面的代码 

```
新建`UEditorController.java`类 用户替换 ueditor自带的jsp 类上传组件
```java

@Controller
@RequestMapping("/ueditor")
public class UEditorController {

	@Value("${ueditor.imagepath}")
	private String imageUploadPath;
	@Value("${ueditor.videopath}")
	private String videoUploadPath;
	@Value("${ueditor.filepath}")
	private String fileUploadPath;
	@Value("${spring.profiles.active}")
	private List<String> active;

	@RequestMapping
	@ResponseBody
	public Object index(String action, HttpServletRequest request, Integer start, Integer size) throws Exception {
		if (null == action)
			return config();
		MultipartHttpServletRequest multipartHttpServletRequest = null;
		MultipartFile upfile;
		if (request instanceof MultipartHttpServletRequest) {
			multipartHttpServletRequest = (MultipartHttpServletRequest) request;
		}
		switch (action) {
		case "config": // 配置获取
			return config();
		case "uploadimage": // 图片上传
			upfile = multipartHttpServletRequest.getFile("upfile");
			return this.upfile(upfile, "image", this.imageUploadPath);
		case "uploadvideo": // 视频上传
			upfile = multipartHttpServletRequest.getFile("upfile");
			return this.upfile(upfile, "video", this.videoUploadPath);
		case "uploadfile": // 附件上传
			upfile = multipartHttpServletRequest.getFile("upfile");
			return this.upfile(upfile, "file", this.fileUploadPath);
		case "uploadscrawl": // 涂鸦上传
			return uploadscrawl(request.getParameter("upfile"));
		case "listimage": // 在线图片获取
			return this.resources(this.imageUploadPath, start, size);
		case "listfile": // 在线文件获取
			return this.resources(this.fileUploadPath, start, size);
		}
		return null;
	}

	/**
	 * 显示图片
	 * 
	 * @param ymd
	 * @param imgname
	 * @param response
	 * @throws IOException
	 */
	@RequestMapping("/image/{ymd}/{imgname}")
	public void image(@PathVariable("ymd") String ymd, @PathVariable("imgname") String imgname,
			HttpServletResponse response) throws IOException {
		this.responsefile(this.imageUploadPath, ymd, imgname, response);
	}

	/**
	 * 显示 视频
	 * 
	 * @param ymd
	 * @param videoname
	 * @param response
	 * @throws IOException
	 */
	@RequestMapping("/video/{ymd}/{videoname}")
	public void video(@PathVariable("ymd") String ymd, @PathVariable("videoname") String videoname,
			HttpServletResponse response) throws IOException {
		this.responsefile(this.videoUploadPath, ymd, videoname, response);
	}

	/**
	 * 文件响应
	 * 
	 * @param ymd
	 * @param filename
	 * @param response
	 * @throws IOException
	 */
	@RequestMapping("/file/{ymd}/{filename}")
	public void file(@PathVariable("ymd") String ymd, @PathVariable("filename") String filename,
			HttpServletResponse response) throws IOException {
		this.responsefile(this.fileUploadPath, ymd, filename, response);
	}

	/**
	 * 文件响应
	 * 
	 * @param filetype
	 * @param ymd
	 * @param filenamne
	 * @param response
	 * @throws IOException
	 */
	private void responsefile(String uploadPathPrefix, String ymd, String filenamne, HttpServletResponse response)
			throws IOException {
		response.setContentType("application/octet-stream;charset=utf-8");
		response.setHeader("content-disposition",
				"attachment;filename=" + java.net.URLEncoder.encode(filenamne, "utf-8"));
		String fullPath = uploadPathPrefix + File.separator + ymd + File.separator + filenamne;
		;
		File downFile = new File(fullPath);
		try (OutputStream os = response.getOutputStream()) {
			try (BufferedInputStream in = new BufferedInputStream(new FileInputStream(downFile))) {
				int len;
				byte[] buf = new byte[4096];
				while ((len = in.read(buf)) != -1) {
					os.write(buf, 0, len);
				}
			}
		}
	}

	/**
	 * 在线资源浏览 分页获取
	 * 
	 * @param resourcepath
	 * @param start
	 * @param size
	 * @return
	 */
	private Map<String, Object> resources(String resourcepath, Integer start, Integer size) {
		Map<String, Object> map = new HashMap<String, Object>();
		map.put("state", "SUCCESS");// UEDI
		map.put("start", start + "");
		File file = new File(resourcepath);
		File[] folders = file.listFiles();
		List<File> folderlist = Arrays.asList(folders);
		folderlist.sort((f1, f2) -> {
			return f2.getName().compareTo(f1.getName());
		});
		List<String> listurl = new ArrayList<String>(0);
		int end = folders.length;
		if (end > 20)
			end = 20;

		for (int i = 0; i < end; i++) {
			File folder = folders[i];
			File[] files = folder.listFiles();
			for (File f : files) {
				String abspath = f.getAbsolutePath();
				String[] fs = abspath.split("\\\\");
				String ymdpath = "/" + fs[fs.length - 4] + "/" + fs[fs.length - 3] + "/" + fs[fs.length - 2] + "/"
						+ fs[fs.length - 1];
				listurl.add(ymdpath);
			}
		}
		map.put("total", listurl.size() + "");
		int len = start + size;
		if (len >= listurl.size())
			len = listurl.size();
		listurl = listurl.subList(start, len);
		List<Map<String, String>> listmap = new ArrayList<Map<String, String>>();
		for (int i = 0; i < listurl.size(); i++) {
			String url = listurl.get(i);
			Map<String, String> urlmap = new HashMap<String, String>();
			urlmap.put("url", url);
			listmap.add(urlmap);
		}
		map.put("list", listmap);
		return map;
	}

	/**
	 * 获取配置
	 * 
	 * @return
	 * @throws IOException
	 */
	private String config() throws IOException {
		BufferedReader reader = null;
		StringBuilder builder = new StringBuilder();
		// 这里根据配置文件来判断 如果似乎部署jar 包的方式需要用到 classLoader 来加载配置文件
		if (null != active && active.contains("prod")) {
			InputStream is = this.getClass().getClassLoader().getResourceAsStream("static/ueditor/jsp/config.json");
			reader = new BufferedReader(new InputStreamReader(is));
		} else {
			reader = new BufferedReader(
					new FileReader(ResourceUtils.getFile("classpath:static/ueditor/jsp/config.json")));
		}
		for (String line = ""; (line = reader.readLine()) != null;) {
			builder.append(line);
		}
		return builder.toString();

	}

	/**
	 * 上传涂鸦图片
	 * 
	 * @param base64img
	 * @return
	 */
	private Map<String, String> uploadscrawl(String base64img) throws IOException {
		String suffix = ".jpg"; // 默认jpg 文件
		String fileName = new SimpleDateFormat("yyyyMMddHHmmssSSS").format(new Date()) + suffix;
		String fullPath = "";
		String ymd = new SimpleDateFormat("yyyyMMdd").format(new Date());
		File floder = new File(this.imageUploadPath + File.separator + ymd);
		if (!floder.exists())
			floder.mkdirs();
		fullPath = floder.getPath() + File.separator + fileName;
		byte[] bs = Base64Utils.decodeFromString(base64img);
		try (OutputStream os = new FileOutputStream(new File(fullPath))) {
			os.write(bs);
			os.flush();
			os.close();
		}
		return ueResult("SUCCESS", String.format("/ueditor/image/%s/%s", ymd, fileName), "", "");
	}

	/**
	 * 上传文件
	 * 
	 * @param upfile   文件
	 * @param filetype 文件类型 image | video | file
	 * @return
	 * @throws IllegalStateException
	 * @throws IOException
	 */
	private Map<String, String> upfile(MultipartFile upfile, String filetype, String uploadPathPrefix)
			throws IllegalStateException, IOException {
		String suffix = upfile.getOriginalFilename().substring(upfile.getOriginalFilename().lastIndexOf("."));
		String fileName = new SimpleDateFormat("yyyyMMddHHmmssSSS").format(new Date()) + suffix;
		String fullPath = "";
		String ymd = new SimpleDateFormat("yyyyMMdd").format(new Date());
		File floder = new File(uploadPathPrefix + File.separator + ymd);
		if (!floder.exists())
			floder.mkdirs();
		fullPath = floder.getPath() + File.separator + fileName;
		upfile.transferTo(new File(fullPath));
		String vmpath = String.format("/ueditor/%s/%s/%s", filetype, ymd, fileName);
		return ueResult("SUCCESS", vmpath, upfile.getName(), upfile.getOriginalFilename());
	}

	/**
	 * 返回 ueditor 参数
	 * 
	 * @param state
	 * @param url
	 * @param title
	 * @param original
	 * @return
	 */
	private Map<String, String> ueResult(String state, String url, String title, String original) {
		Map<String, String> map = new HashMap<String, String>();
		map.put("state", state);// UEDITOR的规则:不为SUCCESS则显示state的内容
		map.put("url", url); // 能访问到你现在图片的路径
		map.put("title", title);
		map.put("original", original);
		return map;
	}

}


```
在application.yml 文件 新增文件保存路径
```
server:
  port: 7001

spring:
  profiles:
    active: dev # 配置启动方式   dev | prod 

# 配置ueditor 下载的位置
ueditor:
  imagepath: D:\\ueditor\\image\\
  videopath: D:\\ueditor\\video\\
  filepath: D:\\ueditor\\file\\
```
## 步骤三 测试 

测试页面代码 
```html
<html>
<head>
<title>富文本编辑器</title>
</head>
<body>
	
	<script id="container" name="content" type="text/plain">
    </script>
	<script type="text/javascript" src="/ueditor/ueditor.config.js"></script>
	<script type="text/javascript" src="/ueditor/ueditor.all.min.js"></script>
	<script type="text/javascript">
		var ue = UE.getEditor('container');
	</script>
</body>
</html>
```

启动springboot 项目打开页面测试即可
