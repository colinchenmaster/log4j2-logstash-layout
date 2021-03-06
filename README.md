[![Build Status](https://secure.travis-ci.org/vy/log4j2-logstash-layout.svg)](http://travis-ci.org/vy/log4j2-logstash-layout)
[![Maven Central](https://img.shields.io/maven-central/v/com.vlkan.log4j2/log4j2-logstash-layout-parent.svg)](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.vlkan.log4j2%22)
[![License](https://img.shields.io/github/license/vy/log4j2-redis-appender.svg)](https://www.gnu.org/licenses/gpl.html)

`LogstashLayout` plugin provides a [Log4j 2.x](https://logging.apache.org/log4j/2.x/)
layout with customizable and [Logstash](https://www.elastic.co/products/logstash)-friendly
JSON formatting.

By default, `LogstashLayout` ships the official `JSONEventLayoutV1` stated by
[log4j-jsonevent-layout](https://github.com/logstash/log4j-jsonevent-layout)
Log4j 1.x plugin. Compared to
[JSONLayout](https://logging.apache.org/log4j/2.x/manual/layouts.html#JSONLayout)
included in Log4j 2.x and `log4j-jsonevent-layout`, `LogstashLayout` provides
the following additional features:

- Additional fields can be added. (See `template` and `templateUri` parameters.)
- JSON schema structure can be customized. (See `template` and `templateUri` parameters.)
- Timestamp formatting can be customized. (See `dateTimeFormatPattern`
  and `timeZoneId` parameters.)

# Usage

Add the `log4j2-logstash-layout` dependency to your POM file

```xml
<dependency>
    <groupId>com.vlkan.log4j2</groupId>
    <artifactId>log4j2-logstash-layout</artifactId>
    <version>${log4j2-logstash-layout.version}</version>
</dependency>
```

together with a valid `log4j-core` dependency:

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j2.version}</version>
</dependency>
```

Below you can find a sample `log4j2.xml` snippet employing `LogstashLayout`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn">
    <Appenders>
        <Console name="CONSOLE" target="SYSTEM_OUT">
            <LogstashLayout dateTimeFormatPattern="yyyy-MM-dd'T'HH:mm:ss.SSSZZZ"
                            templateUri="classpath:LogstashJsonEventLayoutV1.json"
                            prettyPrintEnabled="true"
                            stackTraceEnabled="true"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="CONSOLE"/>
        </Root>
    </Loggers>
</Configuration>
```

This generates an output as follows:

```json
{
  "exception": {
    "exception_class": "java.lang.RuntimeException",
    "exception_message": "test",
    "stacktrace": "java.lang.RuntimeException: test\n\tat com.vlkan.log4j2.logstash.layout.demo.LogstashLayoutDemo.main(LogstashLayoutDemo.java:11)\n"
  },
  "line_number": 12,
  "class": "com.vlkan.log4j2.logstash.layout.demo.LogstashLayoutDemo",
  "@version": 1,
  "source_host": "varlik",
  "message": "Hello, error!",
  "thread_name": "main",
  "@timestamp": "2017-05-25T19:56:23.370+02:00",
  "level": "ERROR",
  "file": "LogstashLayoutDemo.java",
  "method": "main",
  "logger_name": "com.vlkan.log4j2.logstash.layout.demo.LogstashLayoutDemo"
}
```

`LogstashLayout` is configured with the following parameters:

| Parameter Name | Type | Description |
|----------------|------|-------------|
| `prettyPrintEnabled` | boolean | enables pretty-printer (defaults to `false`) |
| `locationInfoEnabled` | boolean | includes the filename and line number in the output (defaults to `false`) |
| `stackTraceEnabled` | boolean | includes stack traces (defaults to `false`) |
| `dateTimeFormatPattern` | String | timestamp formatter pattern (defaults to `yyyy-MM-dd'T'HH:mm:ss.SSSZZZ`) |
| `timeZoneId` | String | time zone id (defaults to `TimeZone.getDefault().getID()`) |
| `mdcKeyPattern` | String | regex to filter MDC keys |
| `ndcPattern` | String | regex to filter NDC items |
| `template` | String | inline JSON template for generating the output (has priority over `templateUri`) |
| `templateUri` | String | JSON template for generating the output (defaults to `classpath:LogstashJsonEventLayoutV1.json`) |

`templateUri` denotes the URI pointing to the JSON template that will be used
while formatting the log events. By default, `LogstashLayout` ships the
[JSONEventLayoutV1](https://github.com/logstash/log4j-jsonevent-layout)
in `LogstashJsonEventLayoutV1.json` within the classpath:

```json
{
  "mdc": "${json:mdc}",
  "ndc": "${json:ndc}",
  "exception": {
    "exception_class": "${json:exceptionClassName}",
    "exception_message": "${json:exceptionMessage}",
    "stacktrace": "${json:exceptionStackTrace}"
  },
  "line_number": "${json:sourceLineNumber}",
  "class": "${json:sourceClassName}",
  "@version": 1,
  "source_host": "${hostName}",
  "message": "${json:message}",
  "thread_name": "${json:threadName}",
  "@timestamp": "${json:timestamp}",
  "level": "${json:level}",
  "file": "${json:sourceFileName}",
  "method": "${json:sourceMethodName}",
  "logger_name": "${json:loggerName}"
}
```

In addition to `${json:<logstash-variable-name>}` lookups, 

In case of need, you can create your own templates with a structure tailored
to your needs. That is, you can add new fields, remove or rename existing
ones, change the structure, etc. Please note that `templateUri` parameter only
supports `file` and `classpath` URI schemes. 

Below is the list of known template variables that will be replaced while
rendering the JSON output.

| Variable Name | Description |
|---------------|-------------|
| `exceptionClassName` | `logEvent.getThrown().getClass().getCanonicalName()` |
| `exceptionMessage` | `logEvent.getThrown().getMessage()` |
| `exceptionStackTrace` | `logEvent.getThrown().printStackTrace()` (inactive when `stackTraceEnabled=false`) |
| `exceptionRootCauseClassName` | the innermost `exceptionClassName` in causal chain |
| `exceptionRootCauseMessage` | the innermost `exceptionMessage` in causal chain |
| `exceptionRootCauseStackTrace` | the innermost `exceptionStackTrace` in causal chain |
| `level` | `logEvent.getLevel()` |
| `loggerName` | `logEvent.getLoggerName()` |
| `mdc` | Mapped Diagnostic Context `Map<String, String>` returned by `logEvent.getContextData()` |
| `message` | `logEvent.getMessage()` |
| `ndc` | Nested Diagnostic Context `String[]` returned by `logEvent.getContextStack()` |
| `sourceClassName` | `logEvent.getSource().getClassName()` |
| `sourceFileName` | `logEvent.getSource().getFileName()` (inactive when `locationInfoEnabled=false`) |
| `sourceLineNumber` | `logEvent.getSource().getLineNumber()` (inactive when `locationInfoEnabled=false`) |
| `sourceMethodName` | `logEvent.getSource().getMethodName()` |
| `threadName` | `logEvent.getThreadName()` |
| `timestamp` | `logEvent.getTimeMillis()` formatted using `dateTimeFormatPattern` and `timeZoneId` |

[Log4j 2.x Lookups](https://logging.apache.org/log4j/2.0/manual/lookups.html)
(e.g., `${java:version}`, `${env:USER}`, `${date:MM-dd-yyyy}`) are supported
in templates too. Though note that while `${json:...}` template variables is
expected to occupy an entire field, that is, `"level": "${json:level}"`, a
lookup can be mixed within a regular text: `"myCustomField": "Hello, ${env:USER}!"`.

See `layout-demo` directory for a sample application using the `LogstashLayout`.

Fat JAR
=======

Project also contains a `log4j2-logstash-layout-fatjar` artifact which
includes all its transitive dependencies in a separate shaded package (to
avoid the JAR Hell) with the exception of `log4j-core`, that you need to
include separately.

This might come handy if you want to use this plugin along with already
compiled applications, e.g., Elasticsearch 5.x, which requires Log4j 2.x.

Contributors
============

- [Yaroslav Skopets](https://github.com/yskopets)

# License

Copyright &copy; 2017 [Volkan Yazıcı](http://vlkan.com/)

log4j2-logstash-layout is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by the Free
Software Foundation, either version 3 of the License, or (at your option) any
later version.

log4j2-logstash-layout is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more
details.

You should have received a copy of the GNU General Public License along with
this program. If not, see http://www.gnu.org/licenses/.
