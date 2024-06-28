**第一**
`  String path = System.getProperty("user.dir")+"upload/";`
**第二**
```
  其中 servletContext 需要注入
  private final ServletContext servletContext;

  String path = servletContext.getRealPath("upload/");
```
**总结**
第一种方法更适合独立的Java应用程序，不依赖Web容器。它使用当前工作目录作为基准路径。
第二种方法专门为Web应用程序设计，能够正确处理Web应用中的资源路径，确保路径与Web应用的结构一致。