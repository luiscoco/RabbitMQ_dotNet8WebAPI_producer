# How to create .NET 8 WebAPI integrating RabbitMQ (message producer)

See also these links:

RabbitMQ and Messaging Concepts (Udemy training): https://www.udemy.com/course/rabbitmq-and-messaging-concepts

See also this github repos:

**Fanout**: https://github.com/luiscoco/RabbitMQ_FanoutDemo

**Direct**: https://github.com/luiscoco/RabbitMQ_DirectDemo

**Topic**: https://github.com/luiscoco/RabbitMQ_TopicDemo

**Headers**: https://github.com/luiscoco/RabbitMQ_HeadersDemo

**Default**: https://github.com/luiscoco/RabbitMQ_DefaultDemo

**ExchangeToExchange**: https://github.com/luiscoco/RabbitMQ_ExchangeToExchangeDemo

**AlternateExchange**: https://github.com/luiscoco/RabbitMQ_AlternateDemo

**PushPull**: https://github.com/luiscoco/RabbitMQ_PushPullDemo

**PriorityQueues**: https://github.com/luiscoco/RabbitMQ_PriorityQueues

**RequestReply-QueueName**: https://github.com/luiscoco/RabbitMQ_RequestReply-QueueName-Demo

**RequestReply**: https://github.com/luiscoco/RabbitMQ_RequestReplyDemo

**RequestReply-MatchingCoding**: https://github.com/luiscoco/RabbitMQ_RequestReply-MatchingCoding-Demo

## What is RabbitMQ? 

It is a **message broker** (an intelligent message bus) that receives messages from producers and routes them to one or more consumers

It is open source, written in Erlang

It is simple and easy to use but powerful

Its features can be extended with plugins

Supports several messaging protocols

AMQP 0-9-1

STOMP 1.0 through 1.2

MQTT 3.1.1

AMQP 1.0

Available on Windows, Linux and Mac

There are also several ready to use Docker images on Docker hub

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/acc0cbc4-17fc-47f9-b89c-3620b60a2cdc)


## 1. Pull and run RabbitMQ Docker container

Install and run **Docker Desktop** on your machine, if you haven't already

You can download Docker from the following link: https://www.docker.com/products/docker-desktop

Start **RabbitMQ** in a Docker container by running the following command:

```
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/25e200c7-7309-4380-97c3-27d269474e6e)

The **password** is also **guest**

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/8f3e7ac0-fb1b-4f5f-9229-189d8d611fbd)

## 2. Create .NET Web API in Visual Studio 2022 or VSCode

To create the project, you can use the **dotnet new** command to create a new **Web API project**

```
dotnet new webapi -o RabbitMQWebAPI
```

## 3. Load project dependencies

First, you will need to install the **RabbitMQ.Client** NuGet package in your project

```
dotnet add package RabbitMQ.Client
```

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/d1754305-a583-4519-9ec3-a55c423d478f)

## 4. Project structure

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/a948c163-84dd-4152-ad96-cff2e02f5cbf)

## 5. Modify application middleware (program.cs)

```csharp
using RabbitMQ.Client;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.OpenApi.Models;

namespace RabbitMQWebAPI_producer
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }

    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            var factory = new ConnectionFactory()
            {
                HostName = "localhost",
                UserName = "guest",
                Password = "guest"
            };
            services.AddSingleton(factory.CreateConnection());

            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "RabbitMQWebAPI", Version = "v1" });
            });
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                app.UseSwagger();
                app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "RabbitMQWebAPI v1"));
            }

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```

For .NET 6 and above versions you can omit the Main and Startup 

**Program.cs**

```csharp
using RabbitMQ.Client;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.OpenApi.Models;

namespace RabbitMQWebAPI_producer
{
    var builder = WebApplication.CreateBuilder(args);

    // Add services to the container.
    builder.Services.AddControllers();

    var factory = new ConnectionFactory()
    {
        HostName = "localhost",
        UserName = "guest",
        Password = "guest"
    };
    builder.Services.AddSingleton(factory.CreateConnection());

    builder.Services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "RabbitMQWebAPI", Version = "v1" });
    });

    var app = builder.Build();

    // Configure the HTTP request pipeline.
    if (app.Environment.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseSwagger();
        app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "RabbitMQWebAPI v1"));
    }

    app.UseRouting();

    app.UseAuthorization();

    app.MapControllers();

    app.Run();
}
```

## 6. Creating the Models

```csharp
﻿namespace RabbitMQWebAPI_producer.Models
{
    public class Value
    {
        public string Text { get; set; }
    }
}
```

## 7. Creating the Controller

```csharp
﻿using RabbitMQ.Client;
using System.Text;
using Microsoft.AspNetCore.Mvc;

namespace RabbitMQWebAPI_producer.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class ValuesController : ControllerBase
    {
        private readonly IConnection _connection;

        public ValuesController(IConnection connection)
        {
            _connection = connection;
        }

        [HttpPost]
        public void Post([FromBody] string value)
        {
            using var channel = _connection.CreateModel();
            channel.QueueDeclare(queue: "hello",
                                 durable: false,
                                 exclusive: false,
                                 autoDelete: false,
                                 arguments: null);

            var body = Encoding.UTF8.GetBytes(value);

            channel.BasicPublish(exchange: "",
                                 routingKey: "hello",
                                 basicProperties: null,
                                 body: body);
        }
    }
}
```

## 8. Running and Testing the application

We build and run the application in Visual Studio 2022 and we send a message to RabbitMQ executing the POST request

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/077e895b-5815-41ca-a1cf-e770d844888e)

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/d0006fdc-4067-4584-bf6e-cf85bde36ade)

Inside RabbitMQ we navigate to queues and we verify we create a new queue called **hello**

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/b8417783-4193-46ef-932b-f6cb1bbb49ed)

We can get the messages inside the queue

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/63bbf713-d986-481d-98b9-4210e8938a6d)

## 9. RabbitMQ Exchange Types

## 9.1. Fanout Exchange

The simplest exchange type, it sends all the messages it receives to all the queues that are bound to it.

It simply ignores the routing information and does not perform any filtering.

Like a postman that photocopies all the mails and puts one copy into each mailbox.

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/f7f941a1-6cb9-4bb0-b6ab-6bbb2bd9f06b)





## 9.2. 

## 9.3. 

## 9.4. 

## 9.5. 

## 9.6. 

## 9.7. 

## 9.8. 

## 9.9. 

## 9.10. 

## 9.11. 

## 9.12. 

