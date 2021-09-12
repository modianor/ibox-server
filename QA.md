
### PrintUtils打印工具类的用法
```java

import static com.anjia.unidbgserver.utils.PrintUtils.*;

// PrintUtils.printFileResolve 用法
@SneakyThrows @Override
public FileResult resolve(Emulator emulator, String pathname, int oflags) {
    // 打印    
    printFileResolve(pathname); // 也可以使用 printFileResolve(pathname,"/path/to/apk/dir/"); 
    // 其他逻辑
    return null;
}

// 会打印出类似代码，可以直接复制到代码里用
//        case "/data/app/xxxxx==/base.apk": {
//            return FileResult.success(new SimpleFileIO(oflags, TempFileUtils.getTempFile("/path/to/apk/dir/"), pathname));
//        }

// PrintUtils.printArgs
@Override 
public long callStaticLongMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
    printArgs("callStaticLongMethodV", signature, vaList);
    // 其他逻辑
    return super.callStaticLongMethodV(vm, dvmClass, signature, vaList);
}
// 会打印出类似代码
// 调用方法名: callStaticLongMethodV,方法签名:XX->xx,参数:0,类型:xxx,值:xxx,参数:1,类型:bb,值:bb
```

### 访问demo返回乱码

正常，因为demo返回的是 `byte[]` 数组，没有对其进行处理 ![](docs/2.png)

### 如何进行单元测试

传统的办法是在一个大类里写各种main函数，每次改了都注释，或者写多个main函数，来回改名字。其实不用这么麻烦，用单元测试就可以了。

具体可以参考  `src/test/java/com/anjia/unidbgserver/service`  下的

- `TTEncryptTest.java` 基本等同于main启动，不加载spring那一套，启动速度快，不测试worker，只测试实际业务

- `TTEncryptWorkerTest` 加载spring整套环境，测试多线程worker

`@Test` 可以简单理解成一个个mian函数，可以直接运行的

`@BeforeEach` 是不管起哪个 `@Test` 方法都会先执行 带`@BeforeEach`

### 修改日志等级

正常运行的，修改 `src/main/resources/logback-spring.xml` 里的

单元测试的，修改 `src/test/resources/logback-test.xml` 里的

### 服务挂掉后如何自动重启

群里有朋友反馈服务运行后会自动挂掉，我没有遇到也没有复现，但是针对这个问题，可以换个思路，将保证服务不挂掉，改成，即使服务挂掉，如何快速自动重启

基于此，给出跨平台的几种方案

跨平台的方案使用docker,支持windows/mac/linux (如何安装docker 参考docker官方文档 https://docs.docker.com/engine/install/)

```bash
docker run --restart=always -d -p9999:9999 anjia0532/unidbg-boot-server
```

windows 下可以用 [nssm](http://www.nssm.cc/download),
参考 [nssm 在windows上部署服务](https://www.cnblogs.com/hai-cheng/p/8670395.html)

linux和mac os 下可以用 Supervisor 参考 [Supervisor-java守护进程工具](https://blog.csdn.net/fuchen91/article/details/107086802/)

### 打包后无法访问文件，报 java.io.FileNotFoundException

以demo里的为例 https://github.com/anjia0532/unidbg-boot-server/blob/main/src/main/java/com/anjia/unidbgserver/service/TTEncrypt.java#L61

把二进制文件 libttEncrypt.so 放到 `src/resources/data/apks/so/` 下, 然后调用`TempFileUtils.getTempFile(LIBTT_ENCRYPT_LIB_PATH)` (
将classpath下的文件copy到临时目录里，后边访问临时目录的即可。)

**为啥不直接写死绝对路径，一旦换机器部署，就要重新修改源代码，重新打包，尤其是把代码分发给网友时，很容易踩坑报错。**

```java

private final static String LIBTT_ENCRYPT_LIB_PATH="data/apks/so/libttEncrypt.so";

vm.loadLibrary(TempFileUtils.getTempFile(LIBTT_ENCRYPT_LIB_PATH),false);
```

### 高并发请求

参考 `com.anjia.unidbgserver.web.TTEncryptController` 和 `com.anjia.unidbgserver.service.TTEncrypt`
和 `com.anjia.unidbgserver.service.TTEncryptWorker` 和 `com.anjia.unidbgserver.service.TTEncryptWorkerTest`

主要unidbg模拟逻辑在 `com.anjia.unidbgserver.service.TTEncrypt` 里

`com.anjia.unidbgserver.web.TTEncryptController` 是暴露给外部http调用的

`com.anjia.unidbgserver.service.TTEncryptWorker` 是用多线程包装了一层

`com.anjia.unidbgserver.service.TTEncryptWorkerTest` 是单元测试

### 修改日志等级

修改 `logback-spring.xml`

例如 `<logger name="com.github.unidbg" level="WARN"/>`,意味着 `com.github.unidbg` 包 及该包名空间以下的只打印`WARN`和`ERROR`

### 升级spring boot或者unidbg版本

修改`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 忽略其他部分 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <!-- spring boot版本号 -->
        <version>2.5.3</version>
        <relativePath/>
    </parent>
    <!-- 忽略其他部分 -->
    <properties>
        <!-- 改成unidbg的版本号 -->
        <unidbg.version>0.9.4</unidbg.version>
        <!-- 忽略其他部分 -->
    </properties>
    <!-- 忽略其他部分 -->
</project>
```