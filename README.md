# How to create .NET 8 WebAPI integrating RabbitMQ (message producer). Default Exchange type.

See also this link:

RabbitMQ and Messaging Concepts (Udemy training): https://www.udemy.com/course/rabbitmq-and-messaging-concepts

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

https://github.com/luiscoco/RabbitMQ_FanoutDemo

The simplest exchange type, it sends all the messages it receives to all the queues that are bound to it.

It simply ignores the routing information and does not perform any filtering.

Like a postman that photocopies all the mails and puts one copy into each mailbox.

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/f7f941a1-6cb9-4bb0-b6ab-6bbb2bd9f06b)

## 9.2. Direct Exchange

https://github.com/luiscoco/RabbitMQ_DirectDemo

Routes messages to the queues based on the "**routing key**" specified in binding definition.

In order to send a message to a queue, routing key on the message and the routing key of the bound queue must be exactly the same.

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/c393c4ec-f886-4f0f-af05-4c8647caf2d8)

## 9.3. Topic Exchange

https://github.com/luiscoco/RabbitMQ_TopicDemo

Topic exchange will perform a wildcard match between the routing key and the routing pattern specified in the binding to publish a message to queue

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/904d6e7e-0934-42d3-8e74-9abe35edd43f)

## 9.4. Headers Exchange

https://github.com/luiscoco/RabbitMQ_HeadersDemo

In RabbitMQ, headers exchanges will use the message header attributes for routing

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/2a32015c-474d-45b7-b869-a009eb59cfa6)

## 9.5. Default Exchange

https://github.com/luiscoco/RabbitMQ_DefaultDemo

When a new queue is created on a RabbitMQ system, it is implicitly bound to a system exchange called "default exchange", with a routing key which is the same as the queue name

Default exchange has no name (empty string)

The type of the default exchange is "direct"

When sending a msessage, if exchange name is left empty, it is handled by the "default exchange"

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/bf251d33-6406-45f8-8aba-155898b27b97)

## 9.6. Exchange to Exchange

https://github.com/luiscoco/RabbitMQ_ExchangeToExchangeDemo

The setup in your diagram includes two exchanges and two queues. Let's examine the message flow:

**Exchange 1**: Appears to be a **direct exchange** where messages are routed to the queues based on a matching routing key

Message 1 with routing key abc goes directly to Queue 1, because there's a binding between Exchange 1 and Queue 1 with that routing key

Message 2 with routing key xyz does not go to any queue from Exchange 1, because there's no binding with that key

**Exchange 2**: Also appears to be a **direct exchange** based on the routing

Message 3 with routing key 123 is not routed to any queue from Exchange 2 because there's no matching binding

Message 2 (presumably the same message that was sent to Exchange 1) and Message 4 both have routing key xyz and are routed to Queue 2, because there's a binding between Exchange 2 and Queue 2 with the routing key xyz

The dashed line from Message 2 indicates that it's being routed to Queue 2 via Exchange 2, not Exchange 1

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/0462b04e-921a-49e8-8f44-6ee6015c4828)

## 9.7. Alternate Exchange

https://github.com/luiscoco/RabbitMQ_AlternateDemo

The concept of an **alternate exchange** is used when you want to handle messages that cannot be routed to any queue

In this case, Exchange 1 is configured with an alternate exchange, Exchange 2

If a message is published to Exchange 1 with a routing key for which there is no matching queue binding, the message will be forwarded to the alternate exchange

There are two exchanges depicted in the diagram:

**Exchange 1 (Direct)**: A direct exchange delivers messages to queues based on the message routing key

A direct exchange will route messages to the queue whose binding key exactly matches the routing key of the message

**Exchange 2 (Fanout)**: A fanout exchange routes messages to all of the queues bound to it, without considering the routing key

It's like a broadcast; every queue gets a copy of the message

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/466529d8-8689-4ef1-8422-5e9c33d0a321)

## 9.8. Push vs Pull

https://github.com/luiscoco/RabbitMQ_PushPullDemo

**Push** 

Consumer application subscribes to the queue and waits for messages

If there is already a message on the queue, or when a new message arrives, it is automatically sent(pushed) to the consumer application

This is the suggested way of getting messages from a queue

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/25e71385-ff94-46b9-a2d5-05cdb5e6f972)

**Pull** 

Consumer application does not subscribe to the queue

But it constantly checks(pulls) the queue for new messages

If there is a message available on the queue, it is manually fetched(pulled) by the consumer application

Even though the pull mode is not recommended, it is the only solution when there is no live connection between message broker and consumer applications

![image](https://github.com/luiscoco/RabbitMQ_dotNet8WebAPI_producer/assets/32194879/32f178e0-c687-46ce-a018-959ab4eb30a6)

## 9.9. Work Queues


## 9.10. Publish - Subscribe


## 9.11. Request - Reply


## 9.12. Priority queues

https://github.com/luiscoco/RabbitMQ_PriorityQueues

## 9.13. 
