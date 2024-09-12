```
package xlike.top.api.utils;

import okhttp3.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Proxy;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Map;
import java.util.concurrent.TimeUnit;

/**
 * @author muchi
 */
public class OkHttpUtils {

    private static final Logger logger = LoggerFactory.getLogger(OkHttpUtils.class);

    // 设置超时时间 30秒 （默认的是10秒）
    private static final int DEFAULT_READ_TIMEOUT = 30;
    private static final int DEFAULT_CONNECT_TIMEOUT = 30;

    // 获取OkHttpClient实例（可自定义超时和代理）
    private static OkHttpClient getClient(Integer connectTimeout, Integer readTimeout, Proxy proxy) {
        OkHttpClient.Builder builder = new OkHttpClient.Builder();

        if (connectTimeout != null) {
            builder.connectTimeout(connectTimeout, TimeUnit.SECONDS);
        } else {
            builder.connectTimeout(DEFAULT_CONNECT_TIMEOUT, TimeUnit.SECONDS);
        }

        if (readTimeout != null) {
            builder.readTimeout(readTimeout, TimeUnit.SECONDS);
        } else {
            builder.readTimeout(DEFAULT_READ_TIMEOUT, TimeUnit.SECONDS);
        }

        if (proxy != null) {
            builder.proxy(proxy);
            logger.info("使用代理服务器 {}:{}", proxy.address().toString(), ((InetSocketAddress) proxy.address()).getPort());
        }

        return builder.build();
    }

    // 通用请求方法
    private static Response sendRequest(String url, String method, Map<String, String> headers, RequestBody body,
                                        Integer connectTimeout, Integer readTimeout, Proxy proxy) throws IOException {
        logger.info("准备发送 {} 请求到 URL: {}", method, url);

        Request.Builder requestBuilder = new Request.Builder().url(url).method(method, body);

        if (headers != null) {
            for (Map.Entry<String, String> header : headers.entrySet()) {
                requestBuilder.addHeader(header.getKey(), header.getValue());
                logger.debug("添加请求头：{} = {}", header.getKey(), header.getValue());
            }
        }

        Request request = requestBuilder.build();
        OkHttpClient client = getClient(connectTimeout, readTimeout, proxy);
        long startTimeMillis  = System.currentTimeMillis();
        LocalDateTime startTime = LocalDateTime.now();
        DateTimeFormatter beginFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        try {
            logger.info("请求开始，当前时间: {}", startTime.format(beginFormatter));
            Response response = client.newCall(request).execute();
            long endTimeMillis = System.currentTimeMillis();
            LocalDateTime endTime = LocalDateTime.now();
            DateTimeFormatter endFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
            logger.info("请求结束，当前时间: {} ,耗时 {} 毫秒，返回状态码: {}", endTime.format(endFormatter), (endTimeMillis - startTimeMillis), response.code());

            return response;
        } catch (IOException e) {
            long errorTimeMillis = System.currentTimeMillis();
            logger.error("请求失败，耗时 {} 毫秒，错误信息: {}", (errorTimeMillis - startTimeMillis), e.getMessage());
            throw e;
        }
    }

    // GET请求（可自定义超时和代理）
    public static Response get(String url, Map<String, String> headers, Integer connectTimeout,
                               Integer readTimeout, Proxy proxy) throws IOException {
        return sendRequest(url, "GET", headers, null, connectTimeout, readTimeout, proxy);
    }

    // GET请求（使用默认超时和无代理）
    public static Response get(String url, Map<String, String> headers) throws IOException {
        return get(url, headers, null, null, null);
    }

    // GET请求（无自定义请求头，使用默认超时和无代理）
    public static Response get(String url) throws IOException {
        return get(url, null);
    }

    // POST请求（可自定义超时和代理）
    public static Response post(String url, Map<String, String> headers, RequestBody body, Integer connectTimeout,
                                Integer readTimeout, Proxy proxy) throws IOException {
        return sendRequest(url, "POST", headers, body, connectTimeout, readTimeout, proxy);
    }

    // POST请求（使用默认超时和无代理）
    public static Response post(String url, Map<String, String> headers, RequestBody body) throws IOException {
        return post(url, headers, body, null, null, null);
    }

    // POST请求（无自定义请求头，使用默认超时和无代理）
    public static Response post(String url, RequestBody body) throws IOException {
        return post(url, null, body);
    }

    // PUT请求（可自定义超时和代理）
    public static Response put(String url, Map<String, String> headers, RequestBody body, Integer connectTimeout,
                               Integer readTimeout, Proxy proxy) throws IOException {
        return sendRequest(url, "PUT", headers, body, connectTimeout, readTimeout, proxy);
    }

    // DELETE请求（可自定义超时和代理）
    public static Response delete(String url, Map<String, String> headers, Integer connectTimeout,
                                  Integer readTimeout, Proxy proxy) throws IOException {
        return sendRequest(url, "DELETE", headers, null, connectTimeout, readTimeout, proxy);
    }

    // 使用代理
    public static Proxy createProxy(String hostname, int port) {
        logger.info("创建代理，主机名: {}，端口: {}", hostname, port);
        return new Proxy(Proxy.Type.HTTP, new InetSocketAddress(hostname, port));
    }

    // 创建JSON请求体
    public static RequestBody createJsonRequestBody(String json) {
        logger.info("创建JSON请求体: {}", json);
        return RequestBody.create(json, MediaType.get("application/json; charset=utf-8"));
    }

    // 创建表单请求体
    public static RequestBody createFormRequestBody(Map<String, String> formData) {
        FormBody.Builder formBuilder = new FormBody.Builder();
        for (Map.Entry<String, String> entry : formData.entrySet()) {
            formBuilder.add(entry.getKey(), entry.getValue());
            logger.debug("添加表单字段：{} = {}", entry.getKey(), entry.getValue());
        }
        return formBuilder.build();
    }

    // 获取响应体字符串
    public static String getResponseBodyAsString(Response response) throws IOException {
        if (response.body() != null) {
            String responseBody = response.body().string();
            logger.debug("响应体内容: {}", responseBody);
            return responseBody;
        }
        return null;
    }

    // 关闭响应
    public static void closeResponse(Response response) {
        if (response != null && response.body() != null) {
            logger.info("关闭响应，URL: {}", response.request().url());
            response.close();
        }
    }
}

```