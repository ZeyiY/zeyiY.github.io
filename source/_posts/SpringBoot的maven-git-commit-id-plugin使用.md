---
title: SpringBoot的maven-git-commit-id-plugin使用
date: 2019-04-09 08:37:37
tags:
---

## Spring boot的maven-git-commit-id-plugin使用

#### 背景

在日常开发部署过程中，您是否遇到以下问题，多人合作时，测试环境不知道部署的哪个分支，测试同学也不清楚如何提bug？线上出现紧急bug，代码也不知道在哪个tag出的问题？是不是我们紧急需要一个接口能够清楚的知道部署的分支是哪个分支，谁部署的呢？这篇文章就是通过一个配置  *Maven git commit id插件*并创建Web服务。在此之后，版本信息会在每次构建过程中自动更新。

###git commit id插件

所有属性的详细信息可以在git commit id插件的[GitHub](https://github.com/ktoso/maven-git-commit-id-plugin)存储库中找到。

1. 添加pom配置，插件部分

```
<plugin>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
    <version>2.2.4</version>
    <executions>
        <execution>
            <id>get-the-git-infos</id>
            <goals>
                <goal>revision</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <dotGitDirectory>${project.basedir}/.git</dotGitDirectory>
        <prefix>git</prefix>
        <verbose>false</verbose>
        <generateGitPropertiesFile>true</generateGitPropertiesFile>
    <generateGitPropertiesFilename>${project.build.outputDirectory}/git.properties</generateGitPropertiesFilename>
        <format>json</format>
        <gitDescribe>
            <skip>false</skip>
            <always>false</always>
            <dirty>-dirty</dirty>
        </gitDescribe>
    </configuration>
</plugin>
```

2. 运行maven build 构建，在targer/classes中，git.properties 文件中添加了JSON格式的版本信息

```
{
  "git.branch" : "master",
  "git.build.host" : "jingshuodeMacBook-Pro-3.local",
  "git.build.time" : "2019-04-07T13:00:13+0800",
  "git.build.user.email" : "",
  "git.build.user.name" : "",
  "git.build.version" : "0.0.1-SNAPSHOT",
  "git.closest.tag.commit.count" : "",
  "git.closest.tag.name" : "",
  "git.commit.id" : "6c6c0fbc1bc18a853d0b9b9361d95e61439f0cac",
  "git.commit.id.abbrev" : "6c6c0fb",
  "git.commit.id.describe" : "6c6c0fb",
  "git.commit.id.describe-short" : "6c6c0fb",
  "git.commit.message.full" : "Merge pull request #4 from nebulacollection/develop\n\nfeat:添加git commit 插件以及接口，便于打包时可以确定当前打包版本",
  "git.commit.message.short" : "Merge pull request #4 from nebulacollection/develop",
  "git.commit.time" : "2019-03-11T00:26:47+0800",
  "git.commit.user.email" : "15763942407@163.com",
  "git.commit.user.name" : "ZeyiY",
  "git.dirty" : "false",
  "git.remote.origin.url" : "https://github.com/nebulacollection/NebulaCollectionApi.git",
  "git.tags" : ""
}
```

3. 版本信息添加到RESTful 接口中

我们要做的就是读取git.properties 返回到接口中

```
@RestController
public class VersionController {

    private static final Logger LOGGER = LoggerFactory.getLogger(VersionController.class);


    @RequestMapping(value = "/version", method = GET)
    public String versionInformation() {
        return readGitProperties();
    }

    private String readGitProperties() {
        try {
            ClassLoader classLoader = getClass().getClassLoader();
            InputStream inputStream = classLoader.getResourceAsStream("git.properties");
            return readFromInputStream(inputStream);
        } catch (IOException e) {
            LOGGER.error("readGitProperties error", e);
            return "Version information could not be retrieved";
        }
    }

    private String readFromInputStream(InputStream inputStream)
            throws IOException {
        StringBuilder resultStringBuilder = new StringBuilder();
        try (BufferedReader br = new BufferedReader(new InputStreamReader(inputStream))) {
            String line;
            while ((line = br.readLine()) != null) {
                resultStringBuilder.append(line).append("\n");
            }
        }
        return resultStringBuilder.toString();
    }
}
```

构建运行项目，在http:localhost:8080/

```
@RestController
public class VersionController {

    private static final Logger LOGGER = LoggerFactory.getLogger(VersionController.class);


    @RequestMapping(value = "/version", method = GET)
    public String versionInformation() {
        return readGitProperties();
    }

    private String readGitProperties() {
        try {
            ClassLoader classLoader = getClass().getClassLoader();
            InputStream inputStream = classLoader.getResourceAsStream("git.properties");
            return readFromInputStream(inputStream);
        } catch (IOException e) {
            LOGGER.error("readGitProperties error", e);
            return "Version information could not be retrieved";
        }
    }

    private String readFromInputStream(InputStream inputStream)
            throws IOException {
        StringBuilder resultStringBuilder = new StringBuilder();
        try (BufferedReader br = new BufferedReader(new InputStreamReader(inputStream))) {
            String line;
            while ((line = br.readLine()) != null) {
                resultStringBuilder.append(line).append("\n");
            }
        }
        return resultStringBuilder.toString();
    }
}
```

构建运行项目，在http:localhost:8080/version 就可以看到构建信息。

