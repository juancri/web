---
title: How to use App.config in .Net Core
date: 2023-06-27
tags:
  - english
  - software
  - development
  - dotnet
  - csharp
layout: layouts/post.njk
---

I have been writing software in [C#](https://en.wikipedia.org/wiki/C_Sharp_(programming_language)) for more than 20 years. It's probably my second favorite language, after [TypeScript](https://www.typescriptlang.org/) (both designed by [Anders Hejlsberg](https://en.wikipedia.org/wiki/Anders_Hejlsberg)).

Enterprise applications have a few common features, including [logging](https://en.wikipedia.org/wiki/Logging_(computing)) and [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection). For these two features, historically I have used [log4net](https://logging.apache.org/log4net/) and [Spring .NET](https://www.springframework.net/), respectively.

[.NET Framework](https://en.wikipedia.org/wiki/.NET_Framework_version_history) was initially launched in 2002 but in the last years it has been replaced by [.NET Core](https://en.wikipedia.org/wiki/.NET) or just .NET (See [.NET Core is the Future of .NET](https://devblogs.microsoft.com/dotnet/net-core-is-the-future-of-net/)). The original .NET Framework used an XML configuration file for each application called [*App.config*](https://learn.microsoft.com/en-us/dotnet/framework/configure-apps/).

Multiple components of the application can define their own structure and retrieve their configuration from this file. For example, the original .Net database access library (called ADO.NET) can retrieve the database connection strings from this App.config file ([source](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/connection-strings-and-configuration-files)):

```xml
<?xml version='1.0' encoding='utf-8'?>
<configuration>
  <connectionStrings>
    <clear />
    <add name="Name"
     providerName="System.Data.ProviderName"
     connectionString="Valid Connection String;" />
  </connectionStrings>
</configuration>
```

Additional complex sections of this file can be defined by loading "configuration sections". Both log4net and Spring .NET used this method to define their own configuration sections and load their configuration from the App.config file. This is a minimalistic version of an App.config file with these two sections defined:

```xml
<?xml version="1.0"?>
<configuration>

	<!-- Here we declare the configuration sections -->
	<configSections>
		<section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net"/>
		<sectionGroup name="spring">
			<section name="context" type="Spring.Context.Support.ContextHandler, Spring.Core" />
			<section name="objects" type="Spring.Context.Support.DefaultSectionHandler, Spring.Core" />
		</sectionGroup>
	</configSections>

	<!-- Config section for log4net -->
	<log4net>
		<appender name="ConsoleAppender" type="log4net.Appender.ConsoleAppender">
			<layout type="log4net.Layout.PatternLayout">
				<conversionPattern value="%date [%thread] %-5level %logger - %message%newline"/>
			</layout>
		</appender>
		</appender>
		<root>
			<level value="DEBUG"/>
			<appender-ref ref="ConsoleAppender"/>
		</root>
	</log4net>

	<!-- Config section for Spring.NET -->
	<spring>
		<context>
			<resource uri="config://spring/objects" />
		</context>
		<objects xmlns="http://www.springframework.net">
			<object name="MyDataAccessClass" type="Test.DataAccessClass">
				<property name="SomeProperty" value="A value" />
			</object>
		</objects>
	</spring>
</configuration>
```

In this file we can identify three parts (see the XML comments). The details of each section are not part of the scope of this post. More details in their documentation: [log4net](https://logging.apache.org/log4net/release/manual/configuration.html) and [Spring.NET](https://www.springframework.net/doc-latest/reference/html/xsd-config.html).

A few days ago I noticed some strange behavior mismatch between two applications running on .Net Core 7. One of them seemed to load the App.config file correctly while the second one, trying to invoke Spring .NET, crash with this exception:

```
Unhandled exception. Spring.Context.ApplicationContextException: No context registered. Use the 'RegisterContext' method or the 'spring/context' section from your configuration file.
   at Spring.Context.Support.ContextRegistry.GetContext()
```

Both applications were fairly similar and both used log4net and Spring .NET. I even noticed that this exception was thrown by one of the applications only when building with `dotnet publish -c release` while `dotnet run` worked just fine.

Googling I found that a [new configuration strategy was introduced for .Net Core](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration) and App.config support was discontinued. I didn't really wanted to change my code so I left that as a final option. My strategy was supported by the fact that one of the applications worked just fine and the second did not, so something weird was happening.

I tried comparing both project files, logic in the main class (where usually both log4net and Spring .NET are initialized before everything else) and even tried removing and adding external dependencies to make both applications as equal as possible.

I finally found that one specific dependency on [Oracle.ManagedDataAccess.Core](https://www.nuget.org/packages/Oracle.ManagedDataAccess.Core) made the difference. When this dependency was present, App.config worked just fine but when I remove it, the app somehow does not load the config file. So I had a workaround: adding `Oracle.ManagedDataAccess.Core` as a dependency even if the app did not need it. Of course, that was not a great solution.

[This question in StackOverflow had the answer](https://stackoverflow.com/questions/45034007/using-app-config-in-net-core), specifically [this comment](https://stackoverflow.com/questions/45034007/using-app-config-in-net-core#comment99110661_45034007) and [this answer](https://stackoverflow.com/a/46482184), which is not the accepted answer.

The most simple workaround is to add [System.Configuration.ConfigurationManager](https://www.nuget.org/packages/System.Configuration.ConfigurationManager) as a dependency. When doing so, App.config will work just fine as it did in the old days. Probably, one of my apps worked just because `System.Configuration.ConfigurationManager` was in the dependency tree of `Oracle.ManagedDataAccess.Core`.

## TL;DR

Add `System.Configuration.ConfigurationManager` as a dependency by running:

```bash
dotnet add package System.Configuration.ConfigurationManager
```

I hope this info can help others so they do not have to spend hours investigating like I had to.
