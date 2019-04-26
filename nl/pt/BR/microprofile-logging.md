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

# Registro
{: #mp-logging}

A abordagem recomendada para criação de log com aplicativos MicroProfile é o padrão de criação de log JSR-47 Java. Comece com as importações a seguir:

```java
import java.util.logging.Level;
import java.util.logging.Logger;
```
{: codeblock}

Segundo, instancie uma instância de um criador de logs no nível de classe:

```java
private static Logger logger = Logger.getLogger(PortfolioService.class.getName());
```
{: codeblock}

Inclua chamadas na instância do `logger` antes, depois e dentro das operações em seu código. Os métodos da interface `Logger` são nomeados para indicar a importância, ou "nível", das informações que estão sendo registradas.

```java
logger.info("Creating portfolio for "+owner);
logger.warning("Unable to send message to JMS provider. Continuing without notification of change in loyalty level.");
```
{: codeblock}

O nível de log é exibido quando essas mensagens são enviadas para o console.

```
[INFO] Creating portfolio for John

[WARNING] Unable to send message to JMS provider. Continuing without notification of change in loyalty level.
```
{: screen}

Os níveis de log fornecem a flexibilidade para escolher dinamicamente quais logs seu aplicativo gravará. Isso permite que você grave o código de log que descreve o estado do aplicativo de alto nível e o conteúdo de depuração detalhado na frente, mas filtre o conteúdo de depuração mais detalhado até que seja necessário. O nível de log `info` é geralmente o nível de saída mínimo, seguido por `fine`, `finer`, `finest` e `debug`.

Quando as entradas de log requerem múltiplas linhas de código ou envolvem operações caras, como concatenação de sequência, considere guardá-las com um teste para determinar se o nível de log está ativado. Isso garante que seu aplicativo não gaste o tempo crucial construindo mensagens de log acabarão apenas sendo filtradas. O exemplo a seguir verifica se o nível de log desejado de `fine` está ativado antes de tentar construir a saída de mensagem.

```java
if (logger.isLoggable(Level.FINE)) {
    StringWriter writer = new StringWriter();
    exception.printStackTrace(new PrintWriter(writer));
    logger.fine(writer.toString());
}
```
{: codeblock}

Para obter mais informações sobre os níveis de log e detalhes de configuração, consulte [Guia de Resolução de Problemas do WebSphere Liberty](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") e a [Documentação da API java.util.logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Criação de log JSON com Liberty
{: #mp-json-logging}

O Liberty suporta a criação de log formatada por JSON. Quando ativadas, as mensagens de log serão gravadas no console no formato JSON. Ative isso usando a sub-rotina de criação de log a seguir em seu `server.xml`:

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

Observe que, enquanto `accessLog` é incluído na lista de origens do console acima, a criação de log de acesso HTTP deve ser ativada antes que esses logs sejam gravados no console. O fragmento a seguir mostra como incluir o subelemento `accessLogging` no elemento `httpEndpoint` em seu `server.xml`:

```xml
<httpEndpoint id="defaultHttpEndpoint" host="\*" httpPort="9080" httpsPort="9443">
  <accessLogging
    filepath="${server.output.dir}/logs/http_defaultEndpoint_access.log"
    logFormat='%h %u %t "%r" %s %b %D %{User-agent}i'>
  </accessLogging>
</httpEndpoint>
```
{: codeblock}

Agora, quando você incluir esse código em seu aplicativo:

```java
if (logger.isLoggable(Level.AUDIT)) {
    logger.audit("Initialization complete");
}
```

você localizará algo como isso nos logs:

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

### Lendo a saída de log JSON
{: #mp-json-log-output}

A saída JSON completa é muito útil para armazenamento de log e procuras, mas não é tão fácil de ler. Talvez seja necessário examinar o conteúdo do log em uma janela do terminal usando `kubectl`. Felizmente, há uma ferramenta de linha de comandos chamada `jq` para ajudar.

`jq` permite que você filtre e se concentre no campo ou campos que você precisa. Por exemplo, se você desejar apenas ver o campo `message` e filtrar todo o restante:

```
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

O Liberty possui algumas mensagens do console primitivas que não são formatadas em JSON. Usar o `grep` assegura que `jq` somente analisará linhas contendo um campo de mensagem.

## Recursos adicionais
{: #mp-log-features}

Diretrizes para usar níveis de log, como quando usar `logger.info` ou `logger.fine`, são algo que cada organização ou projeto tem que tomar decisões. No entanto, esperamos que essas interfaces sejam necessárias e úteis em quase todos os projetos.

Uma melhor prática é usar variáveis de ambiente (alimentadas para o pod por meio de mapas de configuração do Kube ou segredos) em cada campo relevante no `server.xml`. Isso permite mudar a configuração de criação de log sem a necessidade de reconstruir e reimplementar sua imagem do Docker.

Por exemplo, para usar variáveis de ambiente para configurar atributos de criação de log de baixa granularidade, você mudaria a sub-rotina do exemplo anterior:

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

Para algo mais parecido com isso:

```xml
<logging consoleLogLevel="${env.LOG_LEVEL}" consoleFormat="${env.LOG_FORMAT}" consoleSource="${env.LOG_SOURCE}" />
```
{: codeblock}

Outra alternativa é usar a variável de ambiente `WLP_LOGGING_CONSOLE_FORMAT`, conforme descrito em [Documentação de criação de log e rastreio](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"). Isso é semelhante ao exemplo acima: é possível configurar a variável `WLP_LOGGING_CONSOLE_FORMAT` para `basic` (o padrão) ou `json`.

## Painéis do Kibana para o Liberty
{: #liberty-kibana}

Junto com o novo recurso de criação de log JSON, o Liberty fornece painéis do Kibana pré-construídos [que podem ser transferidos por download pelo GitHub](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_icp_json_logging.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"). Siga as instruções no link para instalá-los. Dois novos painéis devem estar disponíveis:

![Painéis Kibana](images/microprofile-logging-image4.png "Painéis Kibana")

Quando você clica no painel para determinação de problema, isso é exibido:

![Detalhes do painel Kibana](images/microprofile-logging-image5.png "Detalhes do painel Kibana")

O painel é interativo. Por exemplo, se você clicar em **INFO** na legenda para o widget **Mensagem do Liberty**, o widget **Procura de mensagens do Liberty** abaixo se filtrará apenas para as mensagens `loglevel=INFO`. O painel federará os dados do log de todos os microsserviços baseados no Liberty, filtrando outros logs do sistema.

Há painéis adicionais do Kibana e do Grafana associados ao gráfico do Helm do Liberty. Eles estão disponíveis como [extensões para o pak de nuvem do Liberty](https://github.com/IBM/charts/tree/master/stable/ibm-websphere-liberty/ibm_cloud_pak/pak_extensions/dashboards){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Próximas etapas
{: #mp-logging-next-steps notoc}

Para obter mais informações sobre como customizar mensagens de log com anexadores, níveis de log e detalhes de configuração, consulte a [Referência oficial do Spring Boot para criação de log](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

Saiba mais sobre como visualizar os logs em cada um dos nossos ambientes de implementação:

* [Logs do Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [{{site.data.keyword.openwhisk}} Logs & Monitoramento](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [Pilha ELK do {{site.data.keyword.cloud_notm}} Private](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")