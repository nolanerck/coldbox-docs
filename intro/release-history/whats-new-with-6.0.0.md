# What's New With 6.0.0

ColdBox 6.0.0 is a major release for the ColdBox HMVC platform.  It has some dramatic new features as we keep pushing for more modern and sustainable approaches to web development. We break down the major areas of development below and you can also find the full release notes per library at the end.

## Engine Deprecation

It is also yet another source code reduction due to the dropping of support for the following CFML Engines:

* Adobe ColdFusion 11
* Luce 4.5

## Release Notes

The full release notes per library can be found below. Just click on the library tab and explore their release notes:

{% tabs %}
{% tab title="ColdBox HMVC" %}
#### Bugs

* \[[COLDBOX-48](https://ortussolutions.atlassian.net/browse/COLDBOX-48)\] - CacheBox creates multiple reap threads if the initial one take longer to complete than the reap frequency
* \[[COLDBOX-339](https://ortussolutions.atlassian.net/browse/COLDBOX-339)\] - Error in AbstractFlashScope: key does't exists due to race conditions
* \[[COLDBOX-822](https://ortussolutions.atlassian.net/browse/COLDBOX-822)\] - InvalidEvent is not working when set to a module event
* \[[COLDBOX-829](https://ortussolutions.atlassian.net/browse/COLDBOX-829)\] - Stopgap for Lucee bug losing sessionCluster application setting
* \[[COLDBOX-850](https://ortussolutions.atlassian.net/browse/COLDBOX-850)\] - XML Converter Updated invoke\(\) to correctly call method by name

#### New Features

* \[[COLDBOX-848](https://ortussolutions.atlassian.net/browse/COLDBOX-848)\] - Improve the bug reporting template for development based on whoops
* \[[COLDBOX-849](https://ortussolutions.atlassian.net/browse/COLDBOX-849)\] - Incorporate Response and RestHandler into core
* \[[COLDBOX-851](https://ortussolutions.atlassian.net/browse/COLDBOX-851)\] - All ColdBox apps get a `coldbox-tasks` scheduler executor for internal ColdBox services and scheduled tasks
* \[[COLDBOX-852](https://ortussolutions.atlassian.net/browse/COLDBOX-852)\] - Updated the default ColdBox config appender to be to console instead of the dummy one
* \[[COLDBOX-853](https://ortussolutions.atlassian.net/browse/COLDBOX-853)\] - ColdBox controller gets a reference to the AsyncManager and registers a new `AsyncManager@coldbox` wirebox mapping
* \[[COLDBOX-855](https://ortussolutions.atlassian.net/browse/COLDBOX-855)\] - Allow for the application to declare it's executors via the new `executors` configuration element
* \[[COLDBOX-856](https://ortussolutions.atlassian.net/browse/COLDBOX-856)\] - Allow for a module to declare it's executors via the new `executors` configuration element
* \[[COLDBOX-858](https://ortussolutions.atlassian.net/browse/COLDBOX-858)\] - Introduction of async/parallel  programming via cbPromises
* \[[COLDBOX-859](https://ortussolutions.atlassian.net/browse/COLDBOX-859)\] - ability to do async scheduled tasks with new async cbpromises

#### Improvements

* \[[COLDBOX-830](https://ortussolutions.atlassian.net/browse/COLDBOX-830)\] - Update cachebox flash ram to standardize on unique key discovery
* \[[COLDBOX-833](https://ortussolutions.atlassian.net/browse/COLDBOX-833)\] - Improvements to threading for interceptors and logging to avoid dumb Adobe duplicates
* \[[COLDBOX-841](https://ortussolutions.atlassian.net/browse/COLDBOX-841)\] - Change announceInterception\(\) and processState\(\) to a single method name like: emit\(\) or announce\(\)
* \[[COLDBOX-846](https://ortussolutions.atlassian.net/browse/COLDBOX-846)\] -  Use relocate and setNextEvent status codes in getStatusCode for testing integration
{% endtab %}

{% tab title="WireBox" %}
#### Bugs

* \[[WIREBOX-90](https://ortussolutions.atlassian.net/browse/WIREBOX-90)\] - Fix constructor injection with virtual inheritance

#### New Features

* \[[WIREBOX-91](https://ortussolutions.atlassian.net/browse/WIREBOX-91)\] - Injector's get a reference to an asyncManager and a task scheduler whether they are in ColdBox or non-ColdBox mode
* \[[WIREBOX-92](https://ortussolutions.atlassian.net/browse/WIREBOX-92)\] - New `executors` dsl so you can easily inject executors ANYWHERE

#### Improvements

* \[[WIREBOX-88](https://ortussolutions.atlassian.net/browse/WIREBOX-88)\] - Improve WireBox error on Adobe CF
{% endtab %}

{% tab title="CacheBox" %}
#### New Features

* \[[CACHEBOX-24](https://ortussolutions.atlassian.net/browse/CACHEBOX-24)\] - CacheBox reaper : migrate to a scheduled task via cbPromises
* \[[CACHEBOX-60](https://ortussolutions.atlassian.net/browse/CACHEBOX-60)\] - CacheFactory gets a reference to an asyncManager and a task scheduler whether they are in ColdBox or non-ColdBox mode
{% endtab %}

{% tab title="LogBox" %}
#### Bugs

* \[[LOGBOX-38](https://ortussolutions.atlassian.net/browse/LOGBOX-38)\] - `Rotate` property is defined but never used

#### New Features

* \[[LOGBOX-5](https://ortussolutions.atlassian.net/browse/LOGBOX-5)\] - Allow config path as string in LogBox init \(standalone\)
* \[[LOGBOX-11](https://ortussolutions.atlassian.net/browse/LOGBOX-11)\] - Allow standard appenders to be configured by name \(instead of full path\)
* \[[LOGBOX-36](https://ortussolutions.atlassian.net/browse/LOGBOX-36)\] - Added an `err()` to abstract appenders for reporting to the error streams
* \[[LOGBOX-42](https://ortussolutions.atlassian.net/browse/LOGBOX-42)\] - All appenders get a reference to the running LogBox instance
* \[[LOGBOX-44](https://ortussolutions.atlassian.net/browse/LOGBOX-44)\] - Rolling appender now uses the new async schedulers to stream data to files

#### Improvements

* \[[LOGBOX-37](https://ortussolutions.atlassian.net/browse/LOGBOX-37)\] - Improvements to threading for logging to avoid dumb Adobe duplicates
* \[[LOGBOX-41](https://ortussolutions.atlassian.net/browse/LOGBOX-41)\] - refactoring of internal utility closures to udfs to avoid ACF memory leaks: CF-420487
{% endtab %}
{% endtabs %}

