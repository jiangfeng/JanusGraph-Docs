27. Elasticsearch
---
Elasticsearch是一个分布式、RESTful搜索和分析引擎，能够解决越来越多的用例。作为弹性堆栈的核心，它集中存储您的数据，以便您能够发现预期的数据并发现意外的数据。 -Elasticsearch概述

JanusGraph支持Elasticsearch作为索引后端。以下是JanusGraph支持的一些Elasticsearch功能：
- 全文：支持所有Text谓词以搜索与给定单词，前缀或正则表达式匹配的文本属性。
- Geo：支持所有Geo谓词，以搜索在给定查询几何体中相交，内部，不相交或包含的地理属性。支持用于索引的点，圆，框，线和多边形。支持圆，框和多边形用于查询点属性和所有形状以查询非点属性。
- 数字范围：支持所有数字比较Compare。
- 灵活配置：支持远程操作和开放式设置自定义。
- 集合：支持索引SET和LIST基数属性。
- 时间：纳秒粒度时间索引。
- 自定义分析器：选择使用自定义分析器。
- Not Query-normal-form：支持Query-normal-form（QNF）以外的查询。JanusGraph的QNF是CNF（联合正常形式）的变体，在可能的情况下内联否定。

有关哪些版本的Elasticsearch可与JanusGraph配合使用的详细信息，请参阅附录B Version Compatibility。

>重点：从Elasticsearch 5.0开始，JanusGraph使用沙盒[Painless scripts](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-painless.html)进行内联更新，默认情况下在Elasticsearch 5.x中启用。
将JanusGraph与Elasticsearch 2.x一起使用需要通过设置script.engine.groovy.inline.update为trueElasticsearch集群来启用Groovy内联脚本（有关更多信息，请参阅[动态脚本文档](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/modules-scripting.html#enable-dynamic-scripting)）。

### 27.1 运行 Elasticsearch
JanusGraph支持与正在运行的Elasticsearch集群的连接。JanusGraph提供了两个选项来运行本地Elasticsearch实例以便快速入门。JanusGraph服务器（参见第7.1节 入门(https://docs.janusgraph.org/latest/server.html#server-getting-started) ”）自动启动本地Elasticsearch实例。另外，JanusGraph版本包含一个完整的Elasticsearch发行版，允许用户手动启动本地Elasticsearch实例,有关详细信息，请参阅此[页面](https://www.elastic.co/guide/en/elasticsearch/guide/current/running-elasticsearch.html)。
```
$ elasticsearch/bin/elasticsearch
```
>注意:  
>出于安全原因，Elasticsearch必须在非root帐户下运行

### 27.2 Elasticsearch配置概述
JanusGraph支持与正在运行的Elasticsearch集群的HTTP(S)客户端连接。有关哪些版本的Elasticsearch将与JanusGraph中的不同客户端类型一起使用的详细信息，请参阅[附录B，版本兼容性](https://docs.janusgraph.org/latest/version-compat.html)。
>注意:  
>JanusGraph的索引选项以字符串“ index.[X].” 开头，其中“ [X]”是后端的用户定义名称。在构建混合索引时，必须将此用户定义的名称传递给JanusGraph的ManagementSystem接口，如[第11.1.2节“混合索引”](https://docs.janusgraph.org/latest/indexes.html#index-mixed)中所述，以便JanusGraph知道可能使用多个已配置的索引后端。本章中的配置片段使用名称搜索，而对选项的详细讨论通常将[X]写在相同的位置。只要在janusgraph的配置中和管理索引时一致地使用索引名，那么确切的索引名就不重要。

>提示：  
>建议索引名称仅包含字母数字小写字符和连字符，并且它们以小写字母开头。

#### 27.2.1 连接到Elasticsearch
Elasticsearch客户端指定如下：
```
index.search.backend=elasticsearch
```
连接到Elasticsearch时，必须提供Elasticsearch实例的单个或主机名列表。这些是通过JanusGraph的index.[X].hostname密钥提供的。
```
index.search.backend=elasticsearch
index.search.hostname=10.0.0.10:9200
```
这里指定的每个主机或主机:端口对将被添加到HTTP客户机的请求目标轮询列表中。下面是一个最小配置，它将在默认的Elasticsearch HTTP端口(9200)和端口7777上轮询10.0.0.10和10.0.0.20:
```
index.search.backend=elasticsearch
index.search.hostname=10.0.0.10, 10.0.0.20:7777
```
##### 27.2.1.1 JanusGraph index.[X]和index.[X].elasticsearch选项
JanusGraph只使用默认值index-name和health-request-timeout。有关这些选项及其可接受值的说明，请参见第15章，[配置参考](https://docs.janusgraph.org/latest/config-ref.html)。
- index.[X].elasticsearch.index-name
- index.[X].elasticsearch.health-request-timeout

#### 27.2.2. REST 客户端选项
REST客户端接受该index.[X].bulk-refresh选项。此选项控制何时使更改对搜索可见。有关更多信息，请参阅？[refresh文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)

#### 27.2.3 REST客户端HTTPS配置
将index.[X].elasticsearch.ssl.enabled选项设置为true来启用HTTP的SSL。注意，根据您的配置，如果您您的HTTPS端口号与REST API(9200)的默认端口号不同)，可能需要更改 index.[X].port。
启用SSL后，您还可以配置信任库的位置和密码。这可以按如下方式完成：
```
index.search.elasticsearch.ssl.truststore.location=/path/to/your/truststore.jks
index.search.elasticsearch.ssl.truststore.password=truststorepwd
```
请注意，这些设置仅适用于Elasticsearch REST客户端，不会影响JanusGraph中的任何其他SSL连接。

还支持客户端密钥库的配置：
```
index.search.elasticsearch.ssl.keystore.location=/path/to/your/keystore.jks
index.search.elasticsearch.ssl.keystore.storepassword=keystorepwd
index.search.elasticsearch.ssl.keystore.keypassword=keypwd
```
任何密码都可以为空。

如果需要，可以通过将index.[X].elasticsearch.ssl.disable-hostname-verification属性值设置为true来禁用SSL主机名验证，并通过将index.[X].elasticsearch.ssl.allow-self-signed-certificates属性值设置为true来启用对自签名SSL证书的支持。

>提示:  
>不建议依赖自签名SSL证书或禁用生产系统的主机名验证，因为这大大限制了客户端向Elasticsearch服务器提供安全通信通道的能力。这可能会导致泄漏机密数据，这些数据可能是JanusGraph索引的一部分。

#### 27.2.4 REST客户端HTTP身份验证

REST客户端支持以下身份验证选项：基本HTTP身份验证（用户名/密码）和基于用户提供的实现的自定义身份验证。  
这些身份验证方法独立于上述SSL客户端身份验证。

##### 27.2.4.1 REST客户端基本HTTP身份验证

无论SSL支持的状态如何，都可以使用基本HTTP身份验证。可选地，可以通过index.[X].elasticsearch.http.auth.basic.realm属性指定认证领域。
```
index.search.elasticsearch.http.auth.type=basic
index.search.elasticsearch.http.auth.basic.username=httpuser
index.search.elasticsearch.http.auth.basic.password=httppassword
```
>提示：
>强烈建议在使用此选项时使用SSL（例如设置index.[X].elasticsearch.ssl.enabled为true），因为在通过未加密的连接发送时可以拦截凭据！

##### 27.2.4.2 REST客户端自定义HTTP身份验证
可以通过提供自己的实现来实现其他身份验证方法。自定义身份验证器配置如下：
```
index.search.elasticsearch.http.auth.custom.authenticator-class=fully.qualified.class.Name
index.search.elasticsearch.elasticsearch.http.auth.custom.authenticator-args=arg1,arg2,...
```
参数列表是可选的，可以为空。

指定的类必须实现org.janusgraph.diskstorage.es.rest.util.RestClientAuthenticator接口或扩展org.janusgraph.diskstorage.es.rest.util.RestClientAuthenticatorBase工具类。该实现可以访问HTTP客户端配置，并可以根据需要自定义客户端。有关更多信息，请参阅附录A，[API文档（JavaDoc）](https://docs.janusgraph.org/latest/javadoc.html)。

例如，以下代码段实现了一个身份验证器，允许Elasticsearch REST客户端进行身份验证并获得针对AWS IAM的授权：
```
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.ZoneOffset;

import org.apache.http.HttpRequestInterceptor;
import org.apache.http.impl.nio.client.HttpAsyncClientBuilder;
import org.janusgraph.diskstorage.es.rest.util.RestClientAuthenticatorBase;

import com.amazonaws.auth.DefaultAWSCredentialsProviderChain;
import com.amazonaws.regions.DefaultAwsRegionProviderChain;
import com.google.common.base.Supplier;

import vc.inreach.aws.request.AWSSigner;
import vc.inreach.aws.request.AWSSigningRequestInterceptor;

/**
 * <p>
 * Elasticsearch REST HTTP(S) client callback implementing AWS request signing.
 * </p>
 * <p>
 * The signer is based on AWS SDK default provider chain, allowing multiple options for providing
 * the caller credentials. See {@link DefaultAWSCredentialsProviderChain} documentation for the details.
 * </p>
 */
public class AWSV4AuthHttpClientConfigCallback extends RestClientAuthenticatorBase {

    private static final String AWS_SERVICE_NAME = "es";
    private HttpRequestInterceptor awsSigningInterceptor;

    public AWSV4AuthHttpClientConfigCallback(final String[] args) {
        // does not require any configuration
    }

    @Override
    public void init() throws IOException {
        DefaultAWSCredentialsProviderChain awsCredentialsProvider = new DefaultAWSCredentialsProviderChain();
        final Supplier<LocalDateTime> clock = () -> LocalDateTime.now(ZoneOffset.UTC);

        // using default region provider chain
        // (http://docs.aws.amazon.com/sdk-for-java/v2/developer-guide/java-dg-region-selection.html)
        DefaultAwsRegionProviderChain regionProviderChain = new DefaultAwsRegionProviderChain();
        final String awsRegion = regionProviderChain.getRegion();

        final AWSSigner awsSigner = new AWSSigner(awsCredentialsProvider, awsRegion, AWS_SERVICE_NAME, clock);
        this.awsSigningInterceptor = new AWSSigningRequestInterceptor(awsSigner);
    }

    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.addInterceptorLast(awsSigningInterceptor);/
    }
}
```
此自定义身份验证器不使用任何构造函数参数。

#### 27.2.5 摄取管道
如果使用Elasticsearch 5.0或更高版本，则可以为每个混合索引设置不同的摄取管道。可以使用摄取管道在索引之前预处理文档。管道由一系列处理器组成。每个处理器以某种方式转换文档。例如，[日期处理器](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-processor.html)可以从文本提取日期到日期字段。因此，您可以使用JanusGraph查询此日期，而不会将其物理存储在主存储中。
- index.[X].elasticsearch.ingest-pipeline.[mixedIndexName] = pipeline_id 

有关摄取处理程序管道和[处理器](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-processors.html)的更多信息，请参见[摄取文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)。

### 27.3 安全的Elasticsearch
Elasticsearch不执行身份验证或授权。Elasticsearch信任可以连接到Elasticsearch的客户端。当Elasticsearch在不安全或公共网络（尤其是Internet）上运行时，应使用某种类型的外部安全性进行部署。这通常通过防火墙，Elasticsearch端口的隧道或使用Elasticsearch扩展（如X-Pack）的组合来完成。Elasticsearch有两个面向客户端的端口需要考虑：
- HTTP REST API，通常在端口9200上
- 本地“传输”协议，通常在端口9300上

客户端使用一个协议/端口或另一个，但不能同时使用。保护HTTP协议端口通常使用防火墙和具有SSL加密和HTTP身份验证的反向代理的组合来完成。有两种方法可以在本机“传输”协议端口上实现安全性：  
除此之外，一些托管的Elasticsearch服务还提供其他身份验证和授权方法。例如，AWS Elasticsearch Service需要使用HTTPS并提供使用基于IAM的访问控制的选项。为此，必须签署发送到此服务的请求。这可以通过使用自定义身份验证器来实现（参见上文）。

隧道Elasticsearch的本地“传输”协议
>这种方法可以通过SSL / TLS隧道（例如通过stunnel），VPN或SSH端口转发来实现。SSL / TLS隧道需要非常简单的设置和监控：隧道的一端或两端需要证书，并且需要配置并连续运行stunnel进程。大多数安全VPN的设置同样重要。一些Elasticsearch服务提供商处理服务器端隧道管理并提供自定义Elasticsearch transport.type以简化客户端设置。

添加防火墙规则，该规则仅允许受信任的客户端在Elasticsearch的本机协议端口上进行连接
>这通常在主机防火墙级别完成。易于配置，但本身安全性非常差。

### 27.4 索引创建选项
JanusGraph支持自定义创建Elasticsearch索引时使用的索引设置。它允许settings在JanusGraph发出的Elasticsearch create index请求中设置对象上的任意键值对。以下是可以使用此机制自定义的Elasticsearch索引设置的非详尽示例：
- index.number_of_replicas
- index.number_of_shards
- index.refresh_interval

通过此机制自定义的设置仅在JanusGraph尝试在Elasticsearch中创建其索引时应用。如果JanusGraph发现其索引已经存在，那么它不会尝试重新创建它，并且这些设置无效。

#### 27.4.1 嵌入Elasticsearch索引创建设置create.ext

Janusgraph迭代所有前缀为index.[x].elasticsearch.create.ext.的属性，其中[x]是索引名，如search。它从每个属性键中去掉前缀。剥离的键的其余部分将被解释为ElasticSearch索引创建设置。未修改与键关联的值。剥离的键和未修改的值作为ElasticSearch创建索引请求中的设置对象的一部分传递，该请求是Janusgraph在ElasticSearch上引导时发出的。这允许在Janusgraph的属性中嵌入任意索引创建设置。下面是一个配置片段示例，它使用create.ext配置机制自定义三个ElasticSearch索引设置：
```
index.search.backend=elasticsearch
index.search.elasticsearch.create.ext.number_of_shards=15
index.search.elasticsearch.create.ext.number_of_replicas=3
index.search.elasticsearch.create.ext.shard.check_on_startup=true
```
上面列出的配置片段利用了Elasticsearch在服务器端实现的假设，即不合格的create index设置密钥具有index.前缀。也可以明确地拼出索引前缀。这是一个JanusGraph配置文件，功能上与上面列出的相同，只是index.索引创建设置前的前缀是显式的：
```
index.search.backend=elasticsearch
index.search.elasticsearch.create.ext.index.number_of_shards=15
index.search.elasticsearch.create.ext.index.number_of_replicas=3
index.search.elasticsearch.create.ext.index.shard.check_on_startup=false
```
>提示  
>create.ext指定索引创建设置的机制与JanusGraph的Elasticsearch配置兼容。

### 27.5 故障排除
#### 27.5.1 连接到远程Elasticsearch集群问题
检查来自JanusGraph节点的HTTP协议端口上是否可以访问Elasticsearch集群节点。通过检查Elasticsearch节点配置日志或使用常规诊断实用程序netstat来检查节点侦听端口。检查JanusGraph配置。

### 27.6 优化Elasticsearch
#### 27.6.1 写优化
对于[批量加载](https://docs.janusgraph.org/latest/bulk-loading.html)或其他写入密集型应用程序，请考虑增加Elasticsearch的刷新间隔。请参阅[此讨论](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html)，了解如何增加刷新间隔及其对写入性能的影响。请注意，更高的刷新间隔意味着图形突变在索引中可用的时间更长。

有关如何在Elasticsearch中提高写入性能的其他建议以及详细说明，请阅读[此博客文章](http://blog.bugsense.com/post/35580279634/indexing-bigdata-with-elasticsearch)。

#### 27.6.2 进一步阅读
- 有关Elasticsearch以及如何设置Elasticsearch集群的更多信息，请参阅Elasticsearch主页和可用文档。