## 1 读取保存图片到本地

#### 1.1 前端

```vue
<template>
  <div>
    <el-upload
      action="http://localhost:8090/upload/getpath"
      list-type="picture-card"
      :on-preview="handlePictureCardPreview"
      :on-remove="handleRemove"
      :on-success="handleSuccess"
    >
      <i class="el-icon-plus"></i>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible">
      <img width="100%" :src="dialogImageUrl" alt="">
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Upload',
  data () {
    return {
      dialogImageUrl: '',
      dialogVisible: false
    }
  },
  methods: {
    handleRemove (file, fileList) {
      console.log(file, fileList)
    },
    handlePictureCardPreview (file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    handleSuccess (response, file, fileList) {
      this.$emit('imgurl', response)
    }
  }
}
</script>

<style scoped>

</style>
```

#### 1.2 后端

```java
@RequestMapping("/getpath")
public String getImgPathAndSaveImg(@RequestParam("file") MultipartFile file) {
    String imgStr = null ;
    try {
        imgStr = UploadUtil.getBase64FromInputStream(file.getInputStream());
    } catch (IOException e) {
        e.printStackTrace();
    }
    try {
        if (imgStr != null) {
            String path = UploadUtil.base64ToImage(imgStr);
            return path;
        }
    } catch (Exception e) {
        e.printStackTrace();
        return "发生错误";
    }
    return "保存失败！";
}
```

#### 1.3 工具类

```java
public class UploadUtil {

    /**
     * 读取文件流转换成base64编码字符串
     * @param in 文件流
     * @return base64编码字符串
     */
    public static String getBase64FromInputStream(InputStream in) {
        // 将图片文件转化为字节数组字符串，并对其进行Base64编码处理
        byte[] data = null;
        // 读取图片字节数组
        try {
            ByteArrayOutputStream swapStream = new ByteArrayOutputStream();
            byte[] buff = new byte[100];
            int rc = 0;
            while ((rc = in.read(buff, 0, 100)) > 0) {
                swapStream.write(buff, 0, rc);
            }
            data = swapStream.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return new String(Base64.encodeBase64(data));
    }

    /**
     * base64保存图片
     * @param imgStr base64 图片编码
     * @return  返回图片存放路径
     * @throws Exception 抛出异常
     */
    public static String base64ToImage(String imgStr) throws Exception
    {
        OutputStream out = null;
        String imgFilePath = "C:/test/" + System.currentTimeMillis() + ".jpg";
        File path = new File("C:/test");
        if (!path.exists())
        {
            path.mkdirs();
        }
        try
        {
            imgStr = imgStr.substring(imgStr.indexOf(',')+1);
            BASE64Decoder decoder = new BASE64Decoder();
            // Base64解码
            byte[] b = decoder.decodeBuffer(imgStr);
            for (int i = 0; i < b.length; ++i)
            {
                if (b[i] < 0)
                {
                    b[i] += 256;
                }
            }
            // 生成jpeg图片
            out = new FileOutputStream(imgFilePath);
            out.write(b);
            out.flush();
            out.close();
        }
        finally
        {
            if (out != null)
            {
                out.close();
            }
        }
        return imgFilePath;
    }
}
```

