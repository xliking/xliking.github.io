**步骤一**

[此处申请免费的KEY](https://app.prodia.com/api)

如 ：b9b42bbb-e5a5-4e55-a71d-xxxxxxxxx

**步骤二**

这是他的[API文档](https://docs.prodia.com/reference/generate)

我们用的是java，所以就得用okhttp和阿里的fastjson

**第三步**

导入Maven依赖

```java
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>2.0.40</version>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.1</version>
</dependency>

```

**第四步**

编写请求工具类

```java
**
 * @author 木池
 */
public class Draw_StableDrawUtil {

    private static final String API_KEY = "自己申请的key";
    private static final String HEAD_TYPE = "Content-Type";
    private static final String HEAD_ACCEPT = "accept";
    private static final String HEAD_VALUE = "application/json";
    private static final String HEADER_X_PRODIA_KEY = "X-Prodia-Key";
    private static final String MODELS_URL = "https://api.prodia.com/v1/sdxl/generate";
    private static final Integer TIMEOUT_MAX = 5;
    private static final String BASE_URL = "https://api.prodia.com/v1/job/%s";
    private static final long POLL_INTERVAL_MS = 3000;


    private static final OkHttpClient CLIENT = new OkHttpClient.Builder()
            .callTimeout(TIMEOUT_MAX, TimeUnit.MINUTES)
            .readTimeout(TIMEOUT_MAX, TimeUnit.MINUTES)
            .build();
    private static final Logger log = LoggerFactory.getLogger(Draw_StableDrawUtil.class);



    public static String generateText(String content) throws IOException {
        JSONObject object = JSONObject.parseObject(content);
        String prompt = (String) object.get("prompt");
        try {
            prompt = TranslatorsUtils.textTrans(prompt);
        } catch (Exception e) {
            log.error(e.getMessage());
        }
        object.put("prompt", prompt);
        String requestBodyJson = object.toJSONString();
        MediaType mediaType = MediaType.parse(HEAD_VALUE);
        Request request = new Request.Builder()
                .url(MODELS_URL)
                .post(RequestBody.create(mediaType, requestBodyJson))
                .addHeader(HEAD_ACCEPT, HEAD_VALUE)
                .addHeader(HEAD_TYPE, HEAD_VALUE)
                .addHeader(HEADER_X_PRODIA_KEY, API_KEY)
                .build();
        try (Response response = CLIENT.newCall(request).execute()) {
            if (response.isSuccessful() && response.body() != null) {
                JSONObject jsonObject = JSONObject.parseObject(response.body().string());
                String jobId = jsonObject.getString("job");
                return getJobDetails(jobId);
            }
            throw new IOException("Request failed: " + response.code() + " " + response.message());
        } catch (InterruptedException e) {
            return e.getMessage();
        }
    }

    /**
     * 获取任务详情
     *
     * @param jobId 任务ID
     * @return 图片地址
     */
    private static String getJobDetails(String jobId) throws IOException, InterruptedException {
        // 构建请求URL
        String url = String.format(BASE_URL, jobId);
        // 构建请求
        Request request = new Request.Builder()
                .url(url)
                .get()
                .addHeader(HEAD_ACCEPT, HEAD_VALUE)
                .addHeader(HEADER_X_PRODIA_KEY, API_KEY)
                .build();
        boolean isComplete = false;
        String imageUrl = null;
        // 开始轮询
        while (!isComplete) {
            // 执行请求并处理响应
            try (Response response = CLIENT.newCall(request).execute()) {
                if (response.isSuccessful() && response.body() != null) {
                    // 读取响应体内容
                    String res = response.body().string();
                    JSONObject jsonObject = JSONObject.parseObject(res);
                    String status = jsonObject.getString("status");
                    // 检查作业状态
                    if ("succeeded".equals(status)) {
                        isComplete = true;
                        imageUrl = jsonObject.getString("imageUrl");
                    } else if ("failed".equals(status)) {
                        throw new IOException("Job failed: " + jobId);
                    }
                } else {
                    throw new IOException("Request failed: " + response.code() + " " + response.message());
                }
            }
            if (!isComplete) {
                Thread.sleep(POLL_INTERVAL_MS);
            }
        }
        return imageUrl;
    }
}
```

**第五步**

配合前端vue使用

前端代码我就直接粘贴了，就不多做解释了

```vue
<template>
  <div id="app">
    <div class="chat-container" style="height: 820px; max-width: 1350px">
      <div class="selection">
        <div class="container">
          <div class="settings2" @click="chat"><span style="color: #c05ee7"><b>切换AI对话</b></span></div>
          <div class="settings2" @click="newChat"><span style="color: #2f8099"><b>新版AI对话</b></span></div>
        </div>
        <div class="container">
          <div class="settings2" @click="Dalle3"><span style="color: #69a142"><b>切换D3绘画</b></span></div>
          <div class="settings2" @click="Midj"><span style="color: #c65930"><b>切换MJ绘画</b></span></div>
        </div>

        <h3>模型选择</h3>
        <el-select v-model="buildObject.model" placeholder="请选择模型">
          <el-option
              v-for="item in models"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>

        <h3>样式预设</h3>
        <el-select v-model="buildObject.style_preset" placeholder="请选择预设">
          <el-option
              v-for="item in presets"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            <span style="float: left">{{ item.label }}</span>
            <span style="float: right; color: #8492a6; font-size: 13px">{{ item.value }}</span>
          </el-option>
        </el-select>

        <div class="block">
          <h3>steps 步骤</h3>
          <el-slider v-model="buildObject.steps" style="width: 70%;margin-left: 15%"></el-slider>
        </div>

        <h3>采样器</h3>
        <el-select v-model="buildObject.sampler" placeholder="请选择采样器">
          <el-option
              v-for="item in samplers"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>

        <el-tooltip placement="top">
          <div slot="content">为获得最佳性能，请使用以下分辨率之一<br/>1024x1024、1152x896、1216x832、1344x768、1536x640、640x1536、768x1344、832x1216</div>
          <h3>图像宽度</h3>
        </el-tooltip>
        <el-select v-model="buildObject.width" placeholder="请选择图片宽度">
          <el-option
              v-for="item in widths"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>

        <el-tooltip placement="top">
          <div slot="content">为获得最佳性能，请使用以下分辨率之一<br/>1024x1024、1152x896、1216x832、1344x768、1536x640、640x1536、768x1344、832x1216</div>
          <h3>图像高度</h3>
        </el-tooltip>
        <el-select v-model="buildObject.height" placeholder="请选择图片高度">
          <el-option
              v-for="item in widths"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>




      </div>

      <div class="chat-window">
        <div class="chat-header">
          StableDiffusion绘画
        </div>
        <div class="chat-messages" id="chatMessages">
          <div v-for="message in messages" :key="message.id" class="message-row"
               :class="{ 'user-row': message.is_user, 'system-row': !message.is_user }">
            <!-- 系统消息头像 -->
            <img v-if="!message.is_user" :src="system" class="avatar system-avatar" alt="System Avatar">
            <!-- 消息内容 -->
            <div class="message" :class="{ 'user-message': message.is_user, 'system-message': !message.is_user }">
              <div v-if="message.loading" class="loading-box">
                <div class="loader"></div> <!-- 简单的加载动画 -->
              </div>
              <div v-else-if="Array.isArray(message.content)">
                <div v-for="(url, index) in message.content" :key="index" class="image-box">
<!--                  <el-image-->
<!--                      style="width: 200px; height: 200px; margin: 5px;"-->
<!--                      :src="url"-->
<!--                      fit="cover"-->
<!--                      @click="handleImageClick(url)"-->
<!--                  ></el-image>-->
                  <a :href="url" target="_blank">{{ url }}</a>
                </div>
              </div>
              <div v-else>
                {{ message.content }}
              </div>
            </div>
            <!-- 用户消息头像 -->
            <img v-if="message.is_user" :src="user" class="avatar user-avatar" alt="User Avatar">
          </div>
        </div>
        <div class="chat-input">
          <input type="text" v-model="newMessage" @keydown.enter="sendMessage" placeholder="请输入想要绘画的物体"
                 class="input-message">
          <button @click="sendMessage" class="send-button">发送</button>
        </div>
      </div>
    </div>

    <el-dialog :visible.sync="dialogVisible">
      <el-image :src="dialogImageUrl" style="width: 100%;"></el-image>
    </el-dialog>
  </div>
</template>

<script>

export default {
  data() {
    return {
      system: require('@/assets/system.png'),
      user: require('@/assets/user.png'),
      newMessage: '',
      messages: [],
      nextMessageId: 1,
      loading: false, // 用于跟踪图片加载状态
      imageUrls: [], // 用于存储从服务器响应获得的图片链接
      dialogImageUrl: '', // 当前点击的图片的URL
      dialogVisible: false, // 控制模态框的显示


      models: [{
        value: 'dreamshaperXL10_alpha2.safetensors [c8afe2ef]',
        label: 'dreamshaperXL10_alpha2.safetensors [c8afe2ef]'
      }, {
        value: 'dynavisionXL_0411.safetensors [c39cc051]',
        label: 'dynavisionXL_0411.safetensors [c39cc051]'
      }, {
        value: 'juggernautXL_v45.safetensors [e75f5471]',
        label: 'juggernautXL_v45.safetensors [e75f5471]'
      }, {
        value: 'realismEngineSDXL_v10.safetensors [af771c3f]',
        label: 'realismEngineSDXL_v10.safetensors [af771c3f]'
      }, {
        value: 'sd_xl_base_1.0.safetensors [be9edd61]',
        label: 'sd_xl_base_1.0.safetensors [be9edd61]'
      },],

      presets: [ { value: '3d-model', label: '3D模型' },
        { value: 'analog-film', label: '胶片' },
        { value: 'anime', label: '动漫' },
        { value: 'cinematic', label: '电影化的' },
        { value: 'comic-book', label: '漫画书' },
        { value: 'digital-art', label: '数字艺术' },
        { value: 'enhance', label: '增强' },
        { value: 'fantasy-art', label: '奇幻艺术' },
        { value: 'isometric', label: '等轴测' },
        { value: 'line-art', label: '线描' },
        { value: 'low-poly', label: '低多边形' },
        { value: 'neon-punk', label: '霓虹朋克' },
        { value: 'origami', label: '折纸' },
        { value: 'photographic', label: '摄影的' },
        { value: 'pixel-art', label: '像素艺术' },
        { value: 'texture', label: '纹理' },
        { value: 'craft-clay', label: '手工陶土' }],

      samplers:[
        { value: 'Euler', label: 'Euler' },
        { value: 'Euler_a', label: 'Euler a' },
        { value: 'LMS', label: 'LMS' },
        { value: 'Heun', label: 'Heun' },
        { value: 'DPM2', label: 'DPM2' },
        { value: 'DPM2_a', label: 'DPM2 a' },
        { value: 'DPM2S_a', label: 'DPM++ 2S a' },
        { value: 'DPM2M', label: 'DPM++ 2M' },
        { value: 'DPMSDE', label: 'DPM++ SDE' },
        { value: 'DPM_fast', label: 'DPM fast' },
        { value: 'DPM_adaptive', label: 'DPM adaptive' },
        { value: 'LMS_Karras', label: 'LMS Karras' },
        { value: 'DPM2_Karras', label: 'DPM2 Karras' },
        { value: 'DPM2_a_Karras', label: 'DPM2 a Karras' },
        { value: 'DPM2S_a_Karras', label: 'DPM++ 2S a Karras' },
        { value: 'DPM2M_Karras', label: 'DPM++ 2M Karras' },
        { value: 'DPMSDE_Karras', label: 'DPM++ SDE Karras' }
      ],

      widths:[
        {label:"1024",value:"1024"},
        {label:"1152",value:"1152"},
        {label:"1216",value:"1216"},
        {label:"1344",value:"1344"},
        {label:"1536",value:"1536"},
        {label:"640",value:"640"},
        {label:"768",value:"768"},
        {label:"832",value:"832"},
      ],

      heights:[
        {label:"1024",value:"1024"},
        {label:"896",value:"896"},
        {label:"832",value:"832"},
        {label:"768",value:"768"},
        {label:"640",value:"640"},
        {label:"1536",value:"1536"},
        {label:"1344",value:"1344"},
        {label:"1216",value:"1216"},
      ],

        buildObject: {
          model: "sd_xl_base_1.0.safetensors [be9edd61]",
          prompt: "",
          negative_prompt: "badly drawn",
          style_preset: "analog-film",
          steps: 25,
          cfg_scale: 7,
          seed: -1,
          sampler: "DPM++ 2M Karras",
          width: 1024,
          height: 1024
        }

    };
  },
  created() {
    this.messages.push({
      loading: false,
      is_user: false,
      id: this.nextMessageId++,
      content: '请描述你想要的图片，怕你们搞黄，请自行在浏览器打开',
    });
    this.messages.push({
      loading: false,
      is_user: false,
      id: this.nextMessageId++,
      content: '最好更改所需要的样式预设哦，默认是胶片，胶片图片质量好，但是加载慢。',
    });
  },
  methods: {

    Dalle3(){
      this.$router.push('/DallThree');
    },
    Midj(){
      this.$router.push('/DrawMidjourney');
    },
    chat(){
      this.$router.push('/ChatGPTALL');
    },
    newChat(){
      window.location.href = 'https://gpt.xlike.top/#/';
    },

    handleImageClick(url) {
      this.dialogImageUrl = url; // 设置当前的图片URL
      this.dialogVisible = true; // 显示模态框
    },
    sendMessage() {

      this.buildObject.prompt = this.newMessage;

      if (this.newMessage.trim() !== '') {
        const userMessageContent = this.newMessage.trim();
        this.messages.push({
          id: this.nextMessageId++,
          content: userMessageContent,
          is_user: true
        });
        // 添加一个加载中的消息
        const loadingMessageId = this.nextMessageId++;
        this.messages.push({
          id: loadingMessageId,
          content: '',
          is_user: false,
          loading: true
        });
        // 清空输入框
        this.newMessage = '';
        // 发送POST请求

        let jsonPayload = JSON.stringify(this.buildObject);
        this.$http.post("/draw/stableDraw", {prompt: jsonPayload}).then(response => {
          const imageUrlList = response.data; // 根据您的实际响应结构调整这里的路径
          // 更新消息，替换“加载中”消息为图片消息
          this.messages = this.messages.map(message => {
            if (message.id === loadingMessageId) {
              return {
                ...message,
                content: imageUrlList,
                loading: false
              };
            }
            return message;
          });
        }).catch(error => {
          console.error("There was an error!", error);
          // 处理错误情况，例如移除加载消息，显示错误提示等
        });
      }
    }
  }
};
</script>

<style>
/* ...之前的样式不变... */

/* 修改了消息行的样式 */
.message-row {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.system-row {
  justify-content: flex-start; /* 系统消息靠左 */
}

.user-row {
  justify-content: flex-end; /* 用户消息靠右 */
}

.message {
  max-width: 60%; /* 最大宽度，防止内容过少时过宽 */
  padding: 10px;
  border-radius: 15px;
  margin-bottom: 10px;
}

.user-message {
  background-color: #007bff;
  color: white;
}

.system-message {
  background-color: #f0f0f0;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

.system-avatar {
  margin-right: 10px;
}

.user-avatar {
  margin-left: 10px;
}

.selection {
  width: 300px;
}
.settings {
  width: 85%;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
  background-color: #f2f2f2;
  border: 1px solid #dcdcdc;
  border-radius: 10px;
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
  padding: 5px;
  font-size: 18px;
  color: #333;
  text-shadow: 0 1px 0 #fff;
  margin-top: 20px;
  margin-left: 6%;
  cursor: pointer;
}
.container {
  display: flex;
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
}
.settings2 {
  width: 39%;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
  background-color: #f2f2f2;
  border: 1px solid #dcdcdc;
  border-radius: 10px;
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
  padding: 5px;
  font-size: 18px;
  color: #333;
  text-shadow: 0 1px 0 #fff;
  margin-top: 20px;
  margin-left: 3%;
  cursor: pointer;
}

</style>
```
