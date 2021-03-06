# WebUI -- Base Web UI

`WebUI` is the <<contract, base>> of the <<implementations, web UIs>> in Apache Spark:

* Active Spark applications

* Spark History Server

* Spark Standalone cluster manager

* Spark on Mesos cluster manager

NOTE: Spark on YARN uses a different web framework for the web UI.

`WebUI` is used as the parent of spark-webui-WebUITab.md#parent[WebUITabs].

[[contract]]
[source, scala]
----
package org.apache.spark.ui

abstract class WebUI {
  // only required methods that have no implementation
  // the others follow
  def initialize(): Unit
}
----

NOTE: `WebUI` is a `private[spark]` contract.

.(Subset of) WebUI Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `initialize`
a| [[initialize]] Used in <<implementations, implementations>> only to let them initialize their web components

NOTE: `initialize` does not add anything special to the Scala type hierarchy but a common name to use across `WebUIs` (that could also be possible without it). In other words, `initialize` does not participate in any design pattern or a type hierarchy.
|===

`WebUI` is a Scala abstract class and cannot be <<creating-instance, created>> directly, but only as one of the <<implementations, web UIs>>.

[[implementations]]
.WebUIs
[cols="1,2",options="header",width="100%"]
|===
| WebUI
| Description

| spark-history-server:HistoryServer.md[HistoryServer]
| [[HistoryServer]] Used in Spark History Server

| `MasterWebUI`
| [[MasterWebUI]] Used in Spark Standalone cluster manager

| `MesosClusterUI`
| [[MesosClusterUI]] Used in Spark on Mesos cluster manager

| spark-webui-SparkUI.md[SparkUI]
| [[SparkUI]] `WebUI` of a Spark application

| `WorkerWebUI`
| [[WorkerWebUI]] Used in Spark Standalone cluster manager
|===

[[boundPort]]
Once <<bind, bound to a Jetty HTTP server>>, `WebUI` is available at an HTTP port (and is used in the <<webUrl, web URL>> as `boundPort`).

[[webUrl]]
`WebUI` is available at a web URL, i.e. `http://[publicHostName]:[boundPort]`. The <<publicHostName, publicHostName>> is...FIXME and the <<boundPort, boundPort>> is the port that the <<bind, port the Jetty HTTP Server bound to>>.

[[internal-registries]]
.WebUI's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| `tabs`
| [[tabs]] spark-webui-WebUITab.md[WebUITabs]

Used when...FIXME

| `handlers`
| [[handlers]] `ServletContextHandlers`

Used when...FIXME

| `pageToHandlers`
| [[pageToHandlers]] `ServletContextHandlers` per spark-webui-WebUIPage.md[WebUIPage]

Used when...FIXME

| `serverInfo`
| [[serverInfo]] Optional `ServerInfo` (default: `None`)

Used when...FIXME

| `publicHostName`
a| [[publicHostName]] Host name of the UI

`publicHostName` is either `SPARK_PUBLIC_DNS` environment variable or `spark.driver.host` configuration property.

Defaults to the following if defined (in order):

. `SPARK_LOCAL_HOSTNAME` environment variable
. Host name of `SPARK_LOCAL_IP` environment variable
. `Utils.findLocalInetAddress`

Used exclusively when `WebUI` is requested for the <<webUrl, web URL>>

| `className`
| [[className]]

Used when...FIXME
|===

[[logging]]
[TIP]
====
Enable `INFO` or `ERROR` logging level for the corresponding loggers of the <<implementations, WebUIs>>, e.g. `org.apache.spark.ui.SparkUI`, to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.ui=INFO
```

Refer to spark-logging.md[Logging].
====

=== [[creating-instance]] Creating WebUI Instance

`WebUI` takes the following when created:

* [[securityManager]] `SecurityManager`
* [[sslOptions]] `SSLOptions`
* [[port]] Port number
* [[conf]] ROOT:SparkConf.md[SparkConf]
* [[basePath]] `basePath` (default: empty)
* [[name]] Name (default: empty)

`WebUI` initializes the <<internal-registries, internal registries and counters>>.

NOTE: `WebUI` is a Scala abstract class and cannot be created directly, but only as one of the <<implementations, implementations>>.

=== [[detachPage]] Detaching Page And Associated Handlers from UI -- `detachPage` Method

[source, scala]
----
detachPage(page: WebUIPage): Unit
----

`detachPage`...FIXME

NOTE: `detachPage` is used when...FIXME

=== [[detachTab]] Detaching Tab And Associated Pages from UI -- `detachTab` Method

[source, scala]
----
detachTab(tab: WebUITab): Unit
----

`detachTab`...FIXME

NOTE: `detachTab` is used when...FIXME

=== [[detachHandler-ServletContextHandler]] Detaching Handler -- `detachHandler` Method

[source, scala]
----
detachHandler(handler: ServletContextHandler): Unit
----

`detachHandler`...FIXME

NOTE: `detachHandler` is used when...FIXME

=== [[detachHandler-String]] Detaching Handler At Path -- `detachHandler` Method

[source, scala]
----
detachHandler(path: String): Unit
----

`detachHandler`...FIXME

NOTE: `detachHandler` is used when...FIXME

=== [[attachPage]] Attaching Page to UI -- `attachPage` Method

[source, scala]
----
attachPage(page: WebUIPage): Unit
----

Internally, `attachPage` creates the path of the spark-webui-WebUIPage.md[WebUIPage] that is `/` (forward slash) followed by the spark-webui-WebUIPage.md#prefix[prefix] of the page.

`attachPage` spark-webui-JettyUtils.md#createServletHandler[creates a HTTP request handler]...FIXME

[NOTE]
====
`attachPage` is used when:

* `WebUI` is requested to <<attachTab, attach a WebUITab>> (the spark-webui-WebUITab.md#pages[WebUIPages] actually)

* spark-history-server:HistoryServer.md#initialize[HistoryServer], Spark Standalone's `MasterWebUI` and `WorkerWebUI`, Spark on Mesos' `MesosClusterUI` are requested to `initialize`
====

=== [[attachTab]] Attaching Tab And Associated Pages to UI -- `attachTab` Method

[source, scala]
----
attachTab(tab: WebUITab): Unit
----

`attachTab` <<attachPage, attaches>> every `WebUIPage` of the input spark-webui-WebUITab.md#pages[WebUITab].

In the end, `attachTab` adds the input `WebUITab` to <<tabs, WebUITab tabs>>.

NOTE: `attachTab` is used when...FIXME

=== [[addStaticHandler]] Attaching Static Handler -- `addStaticHandler` Method

[source, scala]
----
addStaticHandler(resourceBase: String, path: String): Unit
----

`addStaticHandler`...FIXME

NOTE: `addStaticHandler` is used when...FIXME

=== [[attachHandler]] Attaching Handler to UI -- `attachHandler` Method

[source, scala]
----
attachHandler(handler: ServletContextHandler): Unit
----

`attachHandler` simply adds the input Jetty `ServletContextHandler` to <<handlers, handlers>> registry and requests the <<serverInfo, ServerInfo>> to `addHandler` (only if the `ServerInfo` is defined).

`attachHandler` is used when:

* <<implementations, web UIs>> (i.e. spark-history-server:HistoryServer.md#initialize[HistoryServer], Spark Standalone's `MasterWebUI` and `WorkerWebUI`, Spark on Mesos' `MesosClusterUI`, spark-webui-SparkUI.md#initialize[SparkUI]) are requested to initialize

* `WebUI` is requested to <<attachPage, attach a page to web UI>> and <<addStaticHandler, addStaticHandler>>

* [SparkContext](../SparkContext.md) is created

* `HistoryServer` is requested to spark-history-server:HistoryServer.md#attachSparkUI[attachSparkUI]

* Spark Standalone's `Master` and `Worker` are requested to `onStart` (and attach their metrics servlet handlers to the web ui)

=== [[getBasePath]] `getBasePath` Method

[source, scala]
----
getBasePath: String
----

`getBasePath` simply returns the <<basePath, base path>>.

NOTE: `getBasePath` is used exclusively when `WebUITab` is requested for the spark-webui-WebUITab.md#basePath[base path].

=== [[getTabs]] Requesting Header Tabs -- `getTabs` Method

[source, scala]
----
getTabs: Seq[WebUITab]
----

`getTabs` simply returns the <<tabs, registered tabs>>.

NOTE: `getTabs` is used exclusively when `WebUITab` is requested for the spark-webui-WebUITab.md#headerTabs[header tabs].

=== [[getHandlers]] Requesting Handlers -- `getHandlers` Method

[source, scala]
----
getHandlers: Seq[ServletContextHandler]
----

`getHandlers` simply returns the <<handlers, registered handlers>>.

NOTE: `getHandlers` is used when...FIXME

=== [[bind]] Binding UI to Jetty HTTP Server on Host -- `bind` Method

[source, scala]
----
bind(): Unit
----

`bind`...FIXME

NOTE: `bind` is used when...FIXME

=== [[stop]] Stopping UI -- `stop` Method

[source, scala]
----
stop(): Unit
----

`stop`...FIXME

NOTE: `stop` is used when...FIXME
