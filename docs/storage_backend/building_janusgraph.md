# **第41章建立JanusGraph**

若要针对最新版本的JanusGraph进行开发，请依赖于JanusGraph版本。请注意，这些版本是开发版本，因此不稳定，可能会更改。除非您对JanusGraph最新的开发状态感兴趣，否则我们建议使用稳定的JanusGraph版本。

```
<dependency>
   <groupId>org.janusgraph</groupId>
   <artifactId>janusgraph-core</artifactId>
   <version>X.Y.Z-SNAPSHOT</version>
</dependency>
```

检查[主支](https://github.com/JanusGraph/janusgraph/tree/master)最新版本。快照可通过[Sonatype储存库](https://oss.sonatype.org/content/repositories/snapshots/org/janusgraph/).

添加此依赖项时，请确保将以下存储库添加到`pom.xml`:

```
<repository>
  <id>ossrh</id>
  <name>Sonatype Nexus Snapshots</name>
  <url>https://oss.sonatype.org/content/repositories/snapshots</url>
  <releases>
    <enabled>false</enabled>
  </releases>
  <snapshots>
    <enabled>true</enabled>
  </snapshots>
</repository>
```

## 41.2.常见问题

Maven构建导致警告错误

在构建或依赖JanusGraph时，一定要使用maven程序集插件。

 