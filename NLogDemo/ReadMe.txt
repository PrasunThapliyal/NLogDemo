23 June 2021
============

NLog with Dotnet Core 3.1 and ElasticSearch
---------------------------------------

ref: https://stackoverflow.com/questions/66424790/nlog-integration-for-logging-in-asp-net-core-api
	Nlog integration for logging in asp.net core API
ref: (****) https://github.com/NLog/NLog/wiki/Getting-started-with-ASP.NET-Core-3
	Getting started with ASP.NET Core 3
ref: https://dzone.com/articles/writing-net-core-applications-log-into-elasticsear
	Helps partly
	https://github.com/NLog/NLog/issues/2435
	Some troubleshooting - finally got ElasticSearch working using this
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
Step 3: Integrate ElasticSearch
	PS: I dont have elasticsearch running on my windows machine, but lets see if we can open the port at onxv1339 and point there
	Open port 9200 on onxv1339
		(i) 
			[root@onxv1339 ~]# docker ps -a | grep elastic
			f0696aeebd09        artifactory.ciena.com/blueplanet/elasticsearch:1.1.49-es5.6.8                    "supervisord -c /yeti"   12 weeks ago        Up 12 weeks               8080/tcp, 9200/tcp, 9300/tcp                           elasticsearch_1.1.49-es5.6.8-2_0
			[root@onxv1339 ~]#
			[root@onxv1339 ~]# docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' elasticsearch_1.1.49-es5.6.8-2_0
			172.16.0.21
			[root@onxv1339 ~]# docker run --privileged -v /proc:/host/proc -e HOST_PORT=9200 -e DEST_IP=172.16.0.21 -e DEST_PORT=9200 wlan0/redirect:latest
			Adding redirect from 9200 -> 172.16.0.21:9200
			iptables -t nat -A DOCKER ! -i docker0 -p tcp --dport 9200 -j DNAT --to-destination 172.16.0.21:9200
			[root@onxv1339 ~]#
		(ii) Test: On browser (windows machine), type http://onxv1339.ott.ciena.com:9200/
			You should see this response:
				{
					"name": "elasticsearch_0",
					"cluster_name": "BP2",
					"cluster_uuid": "wn1LEBB-QOyMl7hjuxB-fQ",
					"version": {
						"number": "5.6.8",
						"build_hash": "688ecce",
						"build_date": "2018-02-16T16:46:30.010Z",
						"build_snapshot": false,
						"lucene_version": "6.6.1"
					},
					"tagline": "You Know, for Search"
				}
	Install NuGet Package "NLog.Targets.ElasticSearch"
	Edit nlog.config
		(i) Maybe its not necessary to add <extensions> anymore .. try by removing
		(ii) Add new logger rule to write to "elastic" .. the target for this rule may be defined in another file (nlog-targets.config)
		(iii) include file defining target for elasticsearch in the <include file="" /> tag
		(iv) You can add custom fields in ElasticSearch logs, such as Application = "NLogDemo - Weather Forecast".
		This appears in elasticsearch results under path hits.hits[1]._source.Application, if the target defines this as a <field />
	Add new file: nlog-targets.config
		Define <target /> for elastic
		Target for elastic must have uri defined
		uri="http://onxv1339.ott.ciena.com:9200"
		The other important thing is index
		index="nlog_elasticsearch_demo"
	Run the Application
		Check logs here (no auth needed)
		http://onxv1339.ott.ciena.com:9200/nlog_elasticsearch_demo/_search
---------------------------------------
Troubleshooting:
	Getting exception: Error when setting property 'Layout' on 'NLog.Targets.ElasticSearch.Field' Exception: System.ArgumentException: LayoutRenderer cannot be found: 'exception-data'
	Fix: For now, comment out this attribute in <nlog> tag
		throwConfigExceptions="true"
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

