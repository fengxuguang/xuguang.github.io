---
title: Maven插件之git-commit-id-plugin
tags:
  - maven
categories:
  - maven
abbrlink: 3c9f261d
date: 2024-08-30 09:54:48
---

## 引入 git-commit-id-plugin 插件

在 Maven 项目的 pom.xml 中添加插件，如下：

```pom
<plugin>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
    <version>4.0.5</version>
    <executions>
        <execution>
            <id>get-the-git-infos</id>
            <goals>
                <goal>revision</goal>
            </goals>
            <!-- 默认是 initialize 阶段执行, 可以修改为其他阶段 -->
            <phase>initialize</phase>
        </execution>
        <execution>
            <id>validate-the-git-infos</id>
            <goals>
                <goal>validateRevision</goal>
            </goals>
            <!-- 默认是打包阶段执行, 可以修改为其他阶段 -->
            <phase>package</phase>
        </execution>
    </executions>
    <configuration>
        <!-- git 文件记录, 默认是 ${project.basedir}/.git -->
        <dotGitDirectory>${project.basedir}/.git</dotGitDirectory>
        <!-- 属性前缀, 默认是 git, 如: `${configuration-prefix}.commit.id` -->
        <prefix>git</prefix>
        <!-- 默认 false, 构建时打印信息 -->
        <verbose>false</verbose>
        <dateFormatTimeZone>Asia/Shanghai</dateFormatTimeZone>
        <!-- 默认 false, 如果是 true, 会生成 properties 文件 -->
        <generateGitPropertiesFile>true</generateGitPropertiesFile>
        <generateGitPropertiesFilename>./src/main/resources/git.properties</generateGitPropertiesFilename>
        <!-- 文件格式, 默认 properties, 可以使用 json; 若设置为 json, 则还需设置 `<commitIdGenerationModel>full</commitIdGenerationModel>` -->
        <format>properties</format>
        <!-- 可以用作非常强大的版本控制助手, 可以参考: https://git-scm.com/docs/git-describe -->
        <gitDescribe>
            <!-- 默认 false, 如果为 true, 则不使用该配置 -->
            <skip>true</skip>
            <always>false</always>
            <dirty>-dirty</dirty>
        </gitDescribe>
    </configuration>
</plugin>
```

通过 `mvn package` 打包后，会在 `resources`下生成 `git.properties` 文件，在项目启动类中读取配置中的信息打印至控制台。代码如下：

```java
private static void gitCommitInfo() {
    ClassPathResource classPathResource = new ClassPathResource("git.properties");
    try {
        Properties properties = PropertiesLoaderUtils.loadProperties(classPathResource);
        log.info("git.commit.id={}", properties.getProperty("git.commit.id"));
        log.info("git.build.time={}", properties.getProperty("git.build.time"));
        log.info("git.build.version={}", properties.getProperty("git.build.version"));
        log.info("git.build.user.name={}", properties.getProperty("git.build.user.name"));
    } catch (IOException e) {
        log.error("resolver git.properties failed!");
    }
}
```

