# Distributed Observability

It is aimed build a distributed system for storing log, trace and metric data.


## Technologies Used

- OpenTelemetry Collector
- Kafka
- Zookeper
- Elasticsearch
- Logstash
- Kibana
- Jaeger Collector
- Jaeger Query
- Prometheus
- Grafana

  
## Deploy

Run this project to deploy it

```bash
  docker compose up -d
```
## Usage/Examples

### .Net

Add the serilog library to the project.

```bash
  Serilog.Sinks.OpenTelemetry
```

Create a class called ElasticsearchSettings in the project.

```
public class ElasticsearchSettings
{
    public string Url { get; set; }
    public string UserName { get; set; }
    public string Password { get; set; }
    public string IndexName { get; set; }
}
```

Add the following code into your appsetting.json file.

```json
"Elasticsearch": {
    "Url": "http://localhost:4318",
    "UserName": "changeme",
    "Password": "changeme",
    "IndexName": "otel.api"
}
```

Create a class called LoggingExtensions and add the following code.

```csharp
public static class LoggingExtensions
{
    public static Action<HostBuilderContext, LoggerConfiguration>
        ConfigureLogging =>
        (builderContext, loggerConfiguration) =>
        {
            var environment = builderContext.HostingEnvironment;

            var elasticSettings = builderContext.Configuration.GetSection("Elasticsearch").Get<ElasticsearchSettings>();

            loggerConfiguration.ReadFrom.Configuration(builderContext.Configuration)
                               .Enrich.FromLogContext()
                               .Enrich.WithExceptionDetails()
                               .Enrich.WithProperty("Env", environment.EnvironmentName)
                               .Enrich.WithProperty("AppName", environment.ApplicationName);

            loggerConfiguration.WriteTo.OpenTelemetry(cfg =>
            {
                cfg.Endpoint = elasticSettings.Url;
                cfg.Protocol = Serilog.Sinks.OpenTelemetry.OtlpProtocol.HttpProtobuf;
            });
        }
}
```
