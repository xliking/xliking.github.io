好处是，可以绕开前端已经写死的参数或者规定，交给我们自己去按照格式组装参数，达到我们想要的目的

导入Maven依赖，离不开okhttp和json工具包

```JAVA
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

**第一步**

首先打开一个网站，以这个网站 https://twoapi-ui.qiangtu.com/ 为例

点击对话，来到对话窗口，右键点击 **检查**，来到**NetWork**，选中，**Fetch/xhr**

来到对话框，输入文本，点击发送按钮，此时会多一个请求，点击进去

![image-20240628183518233](https://github.com/xliking/xliking.github.io/assets/115143710/97f57003-196e-4e23-98b9-9e01beca2ec6)


点击Header，会发现**URL**和**Method** ，往下面翻，会有一个最重要的**Request Header**

找到 **Authorization**，记住 **Bearer sk-1e49426A5A63Ee3C33256F17EF152C02**

找到 **Content-Type**，记住 **application/json**，这两个都很重要

**第二步**

因为是post请求，他的参数都在**payload**中

我们进去之后，找到他的请求体

```json
{
  "messages": [
    {
      "role": "system",
      "content": "你是强大的智谱AI，快来帮助解决我的问题吧"
    },
    {
      "role": "user",
      "content": "1"
    }
  ],
  "stream": true,
  "model": "gpt-3.5-turbo-1106",
  "temperature": 0.5,
  "presence_penalty": 0,
  "frequency_penalty": 0,
  "top_p": 1
}
```

然后就到了java代码环节

```java
//我们就要定义他们的请求路径，和 找到的Authorization(就是API_KEY)
private static final Integer TIMEOUT_MAX = 5;
private static final String API_URL = "https://twoapi-ui.qiangtu.com/v1/chat/completions";
private static final String API_KEY = "sk-1e49426A5A63Ee3C33256F17EF152C02";
private static final MediaType JSON_MEDIA_TYPE = MediaType.get("application/json; charset=utf-8");

private static final OkHttpClient CLIENT = new OkHttpClient.Builder()
        .callTimeout(TIMEOUT_MAX, TimeUnit.MINUTES)
        .readTimeout(TIMEOUT_MAX, TimeUnit.MINUTES)
        .build();
```

构建请求体，我一般都是用的JSONObject
我这个只是实例并不完整

```java
JSONObject jsonBody = new JSONObject();
jsonBody.put("stream", false);
  JSONArray messages = new JSONArray();
        messages.add(new JSONObject().fluentPut("role", "system").fluentPut("content", "[midjourney] 根据要求绘图"));
        messages.add(new JSONObject().fluentPut("role", "user").fluentPut("content", prompt));
        jsonBody.put("messages", messages);
RequestBody body = RequestBody.create(JSON_MEDIA_TYPE, jsonBody.toJSONString());
```

发送请求

```java
Request request = new Request.Builder()
        .url(API_URL)
        .post(body)
        .addHeader("Authorization", "Bearer " + API_KEY)
        .addHeader("Content-Type", "application/json")
        .build();
  try (Response response = CLIENT.newCall(request).execute()) {
            if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
      // 请求的结果大多数都在  response.body().string() ，极少数 流式请求需要自己找
            System.out.println(response.body().string());
        }
```