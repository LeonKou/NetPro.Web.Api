# NetPro.Web.Api


## Web.Api使用
![.NET Core](https://github.com/LeonKou/NetPro/workflows/.NET%20Core/badge.svg)
 [![NuGet](https://img.shields.io/nuget/v/NetPro.Web.Api.svg)](https://nuget.org/packages/NetPro.Web.Api)

集成了其他所有NetPro组件的功能，引用此组件可方便NetPro组件的统一配置

### 使用

* 修改 `Startup.cs`

```csharp

public class Startup
{
 #region Fields

 private readonly IConfiguration _configuration;
 private readonly IWebHostEnvironment _webHostEnvironment;
 private IEngine _engine;
 private NetProOtion _NetProOtion;

 #endregion

 #region Ctor

 /// <summary>
 /// 
 /// </summary>
 /// <param name="configuration"></param>
 /// <param name="webHostEnvironment"></param>
 public Startup(IConfiguration configuration, IWebHostEnvironment webHostEnvironment)
 {
 	_configuration = configuration;
 	_webHostEnvironment = webHostEnvironment;
 }

 #endregion

 // This method gets called by the runtime. Use this method to add services to the  container.
 /// <summary>
 /// 
 /// </summary>
 /// <param name="services"></param>
 public void ConfigureServices(IServiceCollection services)
 {
 	(_engine, _NetProOtion) = services.ConfigureApplicationServices(_configuration, _webHostEnvironment);
 }

 /// <summary>
 /// 
 /// </summary>
 /// <param name="builder"></param>
 public void ConfigureContainer(ContainerBuilder builder)
 {
 	_engine.RegisterDependencies(builder, _NetProOtion);
 }
 
 // This method gets called by the runtime. Use this method to configure the HTTP request  pipeline.
 /// <summary>
 /// 
 /// </summary>
 /// <param name="app"></param>
 /// <param name="env"></param>
 public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
 {
 	app.ConfigureRequestPipeline();
 }
}
```

* 为了Startup文件干净清爽，建议创建`ApiStartup.cs`文件

此文件继承`INetProStartup`接口，提供了microsoft原生依赖注入能力，所有组件注入放于此 ，Startup.cs将不接受组件注入

* 修改`appsettings.json` 文件

```json

{	
	//数据库ORM建议使用FreeSql，为了便于灵活选择使用适合自己的ORM，框架已剔除内置的NetPro.Dapper
	//apollo配置
	"Apollo": {
		"Enabled": false,
		"AppId": "Leon",
		"MetaServer": "http://192.168.56.98:7078",
		"Cluster": "default",
		"Namespaces": "AppSetting,MicroServicesEndpoint",
		"RefreshInterval": 300000,
		"LocalCacheDir": "apollo/data"
	},
	//响应缓存配置，建议不大于3秒
	"ResponseCacheOption": {
		"Enabled": true,
		"Duration": 3,
		"IgnoreVaryQuery": [ "sign", "timestamp" ]
	},
	//日志配置
	"Serilog": {
		"Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.Async", "Serilog.Sinks.File" ],
		"MinimumLevel": {
			"Default": "Information",
			"Override": {
				"Microsoft": "Debug",
				"System": "Debug",
				"System.Net.Http.HttpClient": "Debug"
			}
		},
		"WriteTo:Async": {
			"Name": "Async",
			"Args": {
				"configure": [
					{ "Name": "Console" }
				]
			}
		},
		"Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ],
		"Properties": {
			"Application": "Netpro"
		}
	},

	"AllowedHosts": "*",
	//框架核心配置
	"NetProOption": {
		"ProjectPrefix": "Leon",
		"ProjectSuffix": "",
		"UseResponseCompression": false,
		"ThreadMinCount": 5,
		"ApplicationName": "",
		"RequestWarningThreshold": 5
	},
	//接口签名防篡改配置
	"VerifySignOption": {		
		"Enable": true,
		"IsDarkTheme":true,
		"IsDebug": false,
		"IsForce": false, //是否强制签名
		"Scheme": "attribute", //attribute;global
		"ExpireSeconds": 60,
		"CommonParameters": {
			"TimestampName": "timestamp",
			"AppIdName": "appid",
			"SignName": "sign"
		},
		"AppSecret": {
			"AppId": {
				"sadfsdf": "sdfsfd"
			}
		},
		"IgnoreRoute": [ "api/ignore/", "" ]
	},
	//swagger配置
	"SwaggerOption": {
		"Enable": true,
		"IsDarkTheme":true,//Swagger黑色主题
		"MiniProfilerEnabled": false,
		"XmlComments": [ "", "" ],
		"RoutePrefix": "swagger",
		"Description": "this is swagger for netcore",
		"Title": "Demo swagger",
		"Version": "first version",
		"TermsOfService": "netcore.com",
		"Contact": {
			"Email": "swagger@netcore.com",
			"Name": "swagger",
			"Url": "swagger@netcore.com"
		},
		"License": {
			"Name": "",
			"Url": ""
		},
		"Headers": [ //swagger默认公共头参数
			{
				"Name": "User",
				"Description": "用户"
			}
		], 
		"Query": [ //swagger默认url公共参数
			{
				"Name": "sign",
				"Description": "签名"
			},
			{
				"Name": "timestamp",
				"Description": "客户端时间戳"
			}
		]
	},
	//中间件健康检查配置
	"HealthChecksUI": {
		"HealthChecks": [
			{
				"Name": "HealthList",
				"Uri": "/health"
			}
		],
		"Webhooks": [],
		"EvaluationTimeOnSeconds": 3600, //检查周期，单位秒
		"MinimumSecondsBetweenFailureNotifications": 60
	},

	"Hosting": {
		"ForwardedHttpHeader": "",
		"UseHttpClusterHttps": false,
		"UseHttpXForwardedProto": false
	},
	//redis配置
	"RedisCacheOption": {
		"Enabled": true,
		"RedisComponent": 1,
		"Password": "szgla.com",
		"IsSsl": false,
		"Preheat": 20,
		"Cluster": true, //集群模式
		"ConnectionTimeout": 20,
		"Endpoints": [
			{
				"Port": 7000,
				"Host": "172.16.127.13"
			},
			{
				"Port": 7000,
				"Host": "172.16.127.15"
			}
		],
		"Database": 0,
		"DefaultCustomKey": "NetPro:",//key前缀
		"PoolSize": 50
	},
	//跨服务访问配置
	"MicroServicesEndpoint": {
		"Example": "http://localhost:5000",
		"Baidu": ""
	},
	//mongodb配置
	"MongoDbOptions": {
		"Enabled": false,
		"ConnectionString": null,
		"Database": -1
	},
	//rabbitmq配置
	"RabbitMq": {
		"HostName": "127.0.0.1",
		"Port": "5672",
		"UserName": "guest",
		"Password": "guest"
	},
	"RabbitMqExchange": {
		"Type": "direct",
		"Durable": true,
		"AutoDelete": false,
		"DeadLetterExchange": "default.dlx.exchange",
		"RequeueFailedMessages": true,
		"Queues": [
			{
				"Name": "myqueue",
				"RoutingKeys": [ "routing.key" ]
			}
		]
	}
}


```

* Controller使用

`Controller`继承`ApiControllerBase`抽象类提供统一响应和简化其他操作，如果不需要默认提供的响应格式也可直接继承ControllerBase

```csharp

	/// <summary>
	///
	/// </summary>
	[Route("api/v1/[controller]")]
	public class WeatherForecastController : ApiControllerBase
	{
		private readonly ILogger _logger;
		private IExampleProxy _userApi { get; set; }

		public WeatherForecastController(ILogger logger
			 ,IExampleProxy userApi)
		{
			_logger = logger;
			_userApi = userApi;
		}
	}
```
