23 June 2021
============

NLog with Dotnet Core 3.1 and ElasticSearch
---------------------------------------

ref: https://stackoverflow.com/questions/66424790/nlog-integration-for-logging-in-asp-net-core-api
	Nlog integration for logging in asp.net core API
ref: (****) https://github.com/NLog/NLog/wiki/Getting-started-with-ASP.NET-Core-3
	Getting started with ASP.NET Core 3
ref:
---------------------------------------
Environment:
	Windows 10 Laptop
	Visual Studio 2019 (dotnet core 3.1)
	Linux machine with docker and ElasticSearch container running (onxv1339)
---------------------------------------
Step 1: Create a default app in VS 2019
	Create new app -> ASP.NET Core Web Application -> NLogDemo
	API -> DotNet Core 3.1, No Auth, HTTPS yes
	Create
---------------------------------------
Step 2: Integrate NLog
	We want to use Microsoft's ILogger<T> interface to log, as usual
	But logs should be routed to NLog

	Follow this: https://github.com/NLog/NLog/wiki/Getting-started-with-ASP.NET-Core-3
		(i) Install the latest:
			NLog.Web.AspNetCore 4.9+
			Update the NLog package if possible
		(ii) Create nlog.config file
			Build Action: Content, Copy to output directory: Copy if newer
		(iii) Program.cs
			NLog.Web.NLogBuilder.ConfigureNLog("nlog.config")
			..
			..
			.ConfigureLogging(logging =>
            {
                logging.ClearProviders();
                logging.SetMinimumLevel(Microsoft.Extensions.Logging.LogLevel.Trace);
            })
            .UseNLog(); 
		(iv) appsettings.json
			Set default log level to "Trace" (from "Information")
		(v) Write logs using ILogger
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------
---------------------------------------

