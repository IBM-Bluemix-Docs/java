---

copyright:
  years: 2019
lastupdated: "2019-04-23"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

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

# JAX-RS によるヘルス・チェック
{: #jaxrs-healthcheck}

前のセクションで説明したように、Kubernetes には、コマンドの実行、TCP または HTTP エンドポイントを介したネットワーク・チェックなど、ヘルス・プローブを統合するための複数の方法があります。 これらのプローブのいずれも Java&trade 言語で実装することは可能ですが、ここでは HTTP プローブと MicroProfile ヘルスの JAX-RS 実装について詳しく説明します。

## readiness JAX-RS エンドポイントの定義
{: #mp-readiness}

最もシンプルな readiness エンドポイントは、次の例のようになります。

```java
@Path("ready")
public class ReadinessEndpoint {
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public Response testReadiness() {
    if ( ! ready ) {
      return Response.status(503).entity("{\"status\":\"DOWN\"}").build();
    }
    return Response.ok("{\"status\":\"UP\"}").build();
  }
}
```
{: codeblock}

WebSphere Liberty にデプロイされるアプリケーション内のこのようなエンドポイントは、追加の作業なしで一定レベルの readiness の動作を実現します。 アプリケーションが実行中になるまで、Liberty はヘルス・エンドポイントへの要求が、適切な 503 応答で失敗します。必要な機能が含まれるよう、この基本的な実装を拡張する必要があります。 例えば、依存関係の注入処理が完了したことを確認する場合は、`@ApplicationScoped` CDI リソースを評価することを検討してください。 プローブが実装されたら、前のセクションで説明したように HTTP プローブ・タイプを構成することでそれを有効にすることができます。

フォールト・トレランスの役割と目的を常に覚えておいてください。マイクロサービスは、ダウンストリーム・サービスとの通信の失敗と中断を処理するよう期待されています。 readiness チェック内に実行可能なフォールバックがない機能のチェックのみを含めてください。

## liveness JAX-RS エンドポイントの定義
{: #jaxrs-liveness}

liveness プローブは、チェック対象を慎重に指定する必要があります。失敗は結果としてプロセスの即時終了につながるからです。 シンプルな liveness エンドポイントは、次のようになります。

```java
@Path("liveness")
public class LivenessEndpoint {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response testLiveness() {
      return Response.ok("{\"status\":\"UP\"}").build();
    }
}
```
{: codeblock}

liveness プローブの実装は、回復不能なプロセス状態を示すプロセス・ローカル条件を調べるために拡張することができます。 ただし、そのような処理を実行できるのであれば、他の要求を実行するだけの十分な正常性がサーバーにあるはずであるという事実に、その種のチェックの難しさがあります。 そのため、「シンプルさを維持する」という原則を、liveness チェックの実装に直ちに適用する必要があります。 プローブ・エンドポイントを有効にするよう liveness ソースをコーディングしたら、最後の章で説明されているように、コンテナー・オーケストレーション・ソースで http liveness エンドポイントを有効にすることができます。

再始動サイクルを回避するために、liveness チェックの initialDelaySeconds 属性は、予想される最長のサーバー始動時間より長くする必要があります。 始動に通常 30 秒かかる Java&trade アプリケーション・サーバーでは、それよりも大きな値 (例えば 60 秒) を選択します。

## MicroProfile Health API の使用
{: #mp-health-api}

[MicroProfile Health API](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") (mpHealth) は、ヘルス・チェックの実装を単純化するフレームワークを定義します。 `mpHealth` API のバージョン 1.0 では、単一のエンドポイントが定義されます。 次のバージョンでは 2 つ提供されます。1 つは liveness 用、もう 1 つは readiness 用です。

MicroProfile Health API を Liberty と一緒に使用するには、`mpHealth` フィーチャーを `server.xml` に追加します。

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

その後、`/health` エンドポイントにアクセスして、ブラウザーでアプリケーションの状況をチェックできます。 すべてが正常であれば、コードはペイロード `{"outcome": "UP"}` を返します。 以下のコードは、MicroProfile Health API を使用してポッドが正常か作動可能かを示す方法を示しています。`call` メソッドを使用することで、開発者はポッドの正常性の状況を示すチェック・アルゴリズムを提供できます。 メモリーをほとんど使い果たした、などは、ポッドの正常性を示す良い指標です。 ダウンストリーム・データベース接続のチェックは、ポッドの readiness をチェックする良い方法かもしれません。 特定のチェックがない場合は、以下を使用してポッドが alive か ready かを示すことができます。

```java
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

import javax.enterprise.context.ApplicationScoped;

@Health
@ApplicationScoped
public class HealthResource implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("service-a").withData("service-a", "ok").up().build();
    }
}
```
{: codeblock}

その後、以下に示すように、Kubernetes に対して `livenessProbe` と `readinessProbe` を指定する必要があります。
```yaml
livenessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health| grep -q service-a"]
  initialDelaySeconds: 60
readinessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health | grep -q service-a"]
  initialDelaySeconds: 40
```
{: codeblock}

前述のように、readiness と liveness を示すためにさまざまなロジックを使用できます。 MicroProfile Health を使用して readiness チェックを指示してから、readiness チェック用の JAX-RS エンドポイントを作成したり、その逆を行ったりすることができます。 MicroProfile Health の次のバージョンでは、2 つのエンドポイントが提供されます。1 つは liveness 用、もう 1 つは readiness 用です。

OpenLiberty には、MicroProfile Health エンドポイントにカスタム・チェックを追加する方法の例を提供する実践ガイド: https://openliberty.io/guides/microprofile-health.html があります。

[MicroProfile ヘルス・チェックの実行](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")で説明されているように、MicroProfile ヘルス・エンドポイントのデフォルトの動作をオーバーライドすることができます。
