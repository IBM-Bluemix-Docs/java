---

copyright:
  years: 2019
lastupdated: "2019-04-04"

keywords: java logging, log level java, debug java, json log java, json log help, kibana liberty, liberty messages

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 日志记录
{: #mp-logging}

使用 MicroProfile 应用程序进行日志记录的建议方法是 Java 的 JSR-47 日志记录标准。首先使用以下 import 语句：

```java
import java.util.logging.Level;
import java.util.logging.Logger;
```
{: codeblock}

其次，在类级别对记录器的实例进行实例化：

```java
private static Logger logger = Logger.getLogger(PortfolioService.class.getName());
```
{: codeblock}

在代码中的操作前、操作后和操作内，添加对 `logger` 实例的调用。`Logger` 接口的方法本身会命名为指示所记录信息的重要性或“级别”。

```java
logger.info("Creating portfolio for "+owner);
logger.warning("Unable to send message to JMS provider. Continuing without notification of change in loyalty level.");
```
{: codeblock}

这些消息输出到控制台时，会显示日志级别。

```
[INFO] Creating portfolio for John

[WARNING] Unable to send message to JMS provider. Continuing without notification of change in loyalty level.
```
{: screen}

通过日志级别，可以灵活地动态选择应用程序将写入的日志。这使您能够预先编写用于描述高级别应用程序状态和详细调试内容的日志代码，但过滤掉更详细的调试内容，除非您需要这些内容。日志级别 `info` 通常是最低输出级别，之后的级别由低到高依次为 `fine`、`finer`、`finest` 和 `debug`。

当日志条目需要多行代码或涉及字符串并置等大开销操作时，您应该考虑使用测试来对其进行防护，以确定是否启用了日志级别。这将确保应用程序不会花费关键的时间来构建最终只会被过滤掉的日志消息。以下示例验证目标日志级别 `fine` 是否在尝试构建消息输出之前就已启用。

```java
if (logger.isLoggable(Level.FINE)) {
    StringWriter writer = new StringWriter();
    exception.printStackTrace(new PrintWriter(writer));
    logger.fine(writer.toString());
}
```
{: codeblock}

有关日志级别和配置详细信息的更多信息，请参阅 [WebSphere Liberty 故障诊断指南](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 和 [java.util.logging API 文档](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

## 使用 Liberty 进行 JSON 日志记录
{: #mp-json-logging}

Liberty 支持 JSON 格式的日志记录。启用时，日志消息将以 JSON 格式写入控制台。使用 `server.xml` 中的以下 logging 节可启用此项：

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

请注意，虽然上面的控制台源列表中包含 `accessLog`，但必须启用 HTTP 访问日志记录后，才能将这些日志写入控制台。以下片段说明了如何在 `server.xml` 中将 `accessLogging` 子元素添加到 `httpEndpoint` 元素：

```xml
<httpEndpoint id="defaultHttpEndpoint" host="\*" httpPort="9080" httpsPort="9443">
  <accessLogging
    filepath="${server.output.dir}/logs/http_defaultEndpoint_access.log"
    logFormat='%h %u %t "%r" %s %b %D %{User-agent}i'>
  </accessLogging>
</httpEndpoint>
```
{: codeblock}

现在，将以下代码添加到应用程序时：

```java
if (logger.isLoggable(Level.AUDIT)) {
    logger.audit("Initialization complete");
}
```

您将在日志中找到类似于下面的内容：

```json
{ "type":"liberty_message",
  "host":"trader-54b4d579f7-4zvzk",
  "ibm_userDir":"\/opt\/ol\/wlp\/usr\/",
  "ibm_serverName":"defaultServer",
  "ibm_datetime":"2018-06-21T19:23:21.356+0000",
  "ibm_threadId":"00000028",
  "module":"com.trader.Main",
  "loglevel":"AUDIT",
  "message":"Initialization complete"}
```
{: codeblock}

### 读取 JSON 日志输出
{: #mp-json-log-output}

完整的 JSON 输出对于日志存储和搜索非常有用，但要读取却并不容易。您可能需要在终端窗口中使用 `kubectl` 来检查日志的内容。幸好，有一个名为 `jq` 的命令行工具可提供帮助。

`jq` 允许您过滤掉不需要的字段，而只专注于所需的字段。例如，如果要仅查看 `message` 字段，而过滤掉其他所有内容：

```
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Liberty 具有一些非 JSON 格式的原始控制台消息。使用 `grep` 可确保 `jq` 只解析包含 message 字段的行。

## 其他功能
{: #mp-log-features}

针对使用日志级别（例如，何时使用 `logger.info` 或 `logger.fine`）的准则是每个组织或项目必须要对其做出决策的内容。但是，我们预期这些接口在几乎任何项目中都是必需且有用的。

一个最佳做法是在 `server.xml` 的每个相关字段中使用环境变量（通过 Kube 配置映射或私钥传递给 pod）。这允许您更改日志记录配置，而无需重新构建和重新部署 Docker 映像。

例如，要使用环境变量来设置细颗粒度日志记录属性，可将先前示例中的以下节：

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

更改为类似于以下内容：

```xml
<logging consoleLogLevel="${env.LOG_LEVEL}" consoleFormat="${env.LOG_FORMAT}" consoleSource="${env.LOG_SOURCE}" />
```
{: codeblock}

另一个替代方法是使用 `WLP_LOGGING_CONSOLE_FORMAT` 环境变量，如[日志记录和跟踪文档](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中所述。这类似于上面的示例：可以将 `WLP_LOGGING_CONSOLE_FORMAT` 变量设置为 `basic`（缺省值）或 `json`。

## Liberty 的 Kibana 仪表板
{: #liberty-kibana}

除了新的 JSON 日志记录功能，Liberty 还提供了[可从 GitHub 下载](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_icp_json_logging.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 的预构建 Kibana 仪表板。请遵循链接中的指示信息进行安装。应该会提供两个新仪表板：

![Kibana 仪表板](images/microprofile-logging-image4.png "Kibana 仪表板"){: caption="图 1. Kibana 仪表板" caption-side="bottom"}

单击仪表板以进行问题确定时，可以看到以下内容：

![Kibana 仪表板详细信息](images/microprofile-logging-image5.png "Kibana 仪表板详细信息"){: caption="图 2. Kibana 仪表板详细信息" caption-side="bottom"}

此仪表板是交互式的。例如，如果在 **Liberty 消息**窗口小部件的图注中单击 **INFO**，下方的 **Liberty 消息搜索**窗口小部件会自行进行过滤，以仅显示 `loglevel=INFO` 的消息。此仪表板将联合所有基于 Liberty 的微服务的日志数据，并过滤掉其他系统日志。

还有与 Liberty Helm chart 关联的其他 Kibana 和 Grafana 仪表板。这些仪表板作为 [Liberty Cloud Pak 的扩展](https://github.com/IBM/charts/tree/master/stable/ibm-websphere-liberty/ibm_cloud_pak/pak_extensions/dashboards){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 提供。

## 后续步骤
{: #mp-logging-next-steps notoc}

有关使用附加器、日志级别和配置详细信息来定制日志消息的更多信息，请参阅官方的 [Spring Boot 日志记录参考指南](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

了解有关查看每个部署环境中日志的详细信息：

* [Kubernetes 日志](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [{{site.data.keyword.openwhisk}} 日志和监视](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private ELK 堆栈](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
