**第一步**
xxl-job，有单独的数据库，把对应的表导入进去
**第二步**
编写springboot中的配置文件
```
xxl:
  enable: true  # 是否启用 XXL-JOB，true 表示启用，false 表示禁用
  job:
    admin:
      addresses: http://xxl-job-xxx:8080/xxl-job-admin/  # XXL-JOB 管理中心的地址，用于任务的管理和调度
    executor:
      app-name: financial-data-center  # 执行器的应用名称，用于标识具体的执行器
      access-token: xxxxxx  # 访问令牌，用于执行器与调度中心之间的身份认证
      log-path: ./logs/xxlJob  # 执行器日志的存储路径，相对于当前工作目录
      log-retention-days: 7  # 日志的保留天数，超过这个天数的日志将被自动清理
```
**第三步**
编写xxl-job的配置文件，通过读取配置文件中的相关参数，创建并初始化一个 XxlJobSpringExecutor 实例
```
/**
 * xxl job配置
 * @author 木池
 */
@Component
@Slf4j
@ConditionalOnProperty(prefix = "xxl", name = "enable", havingValue = "true", matchIfMissing = true)
public class XxlJobConfig {

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.executor.app-name}")
    private String appName;

    @Value("${xxl.job.executor.log-retention-days}")
    private Integer logRetentionDay;

    @Value("${xxl.job.executor.access-token}")
    private String accessToken;

    @Value("${xxl.job.executor.log-path}")
    private String logPath;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        log.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appName);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDay);
        return xxlJobSpringExecutor;
    }
}

```
**第四步**
编写xxl-job的定时任务
```
/**
 * @author 木池
 */
@Component
@AllArgsConstructor
public class XxlJobTask {
    @XxlJob("cs")
    public ReturnT<String> cs() {
        LOGGER.info("测试");
        return ReturnT.SUCCESS;
    }
}
```
**另提 - 如何获取xxl-job的多个参数**
```
  String param = XxlJobHelper.getJobParam();
  String[] methodParams = param.split(",");
```