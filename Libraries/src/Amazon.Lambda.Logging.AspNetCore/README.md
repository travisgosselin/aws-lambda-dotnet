# Amazon.Lambda.Logging.AspNetCore

This package contains an implementation of ASP.NET Core's ILogger class, allowing an application to use the standard ASP.NET Core logging functionality to write CloudWatch Log events.

# Configuration

Lambda logging can be configured through code or using a file configuration loaded through the `IConfiguration` interface.
The two below examples set the same logging options, but do it either through code or a JSON config file. 

## Configuration through code

```
public void Configure(ILoggerFactory loggerFactory)
{
    // Create and populate LambdaLoggerOptions object
    var loggerOptions = new LambdaLoggerOptions();
    loggerOptions.IncludeCategory = false;
    loggerOptions.IncludeLogLevel = false;
    loggerOptions.IncludeNewline = true;

    // Configure Filter to only log some 
    loggerOptions.Filter = (category, logLevel) =>
    {
        // For some categories, only log events with minimum LogLevel
        if (string.Equals(category, "Default", StringComparison.Ordinal))
        {
            return (logLevel >= LogLevel.Debug);
        }
        if (string.Equals(category, "Microsoft", StringComparison.Ordinal))
        {
            return (logLevel >= LogLevel.Information);
        }

        // Log everything else
        return true;
    };

    // Configure Lambda logging
    loggerFactory
        .AddLambdaLogger(loggerOptions);
}
```

## Configuration through IConfiguration

Configuration file, `appsettings.json`:
```
{
  "Lambda.Logging": {
    "IncludeCategory": false,
    "IncludeLogLevel": false,
    "IncludeNewline":  true,
    "LogLevel": {
      "Default": "Debug",
      "Microsoft": "Information"
    }
  }
}
```

Creating `LambdaLoggerOptions` from the configuration file:
```
public void Configure(ILoggerFactory loggerFactory)
{
    var configuration = new ConfigurationBuilder()
        .AddJsonFile(APPSETTINGS_PATH)
        .Build();

    var loggerOptions = new LambdaLoggerOptions(configuration);
    var loggerfactory = new TestLoggerFactory()
        .AddLambdaLogger(loggerOptions);

    // Configure Lambda logging
    loggerFactory
        .AddLambdaLogger(loggerOptions);
}
```
