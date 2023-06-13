# Publish and Subscribe Solution with Kafka and .NET                  
       
This guide walks you through the process of creating a simple publish and subscribe solution using Apache Kafka and .NET. The solution demonstrates how to use the Confluent.Kafka library to publish messages to a Kafka topic and subscribe to that topic to consume the messages.
  
## Prerequisites

Before you begin, ensure that you have the following prerequisites set up:

- Apache Kafka installed and running locally. You can follow the official [Kafka Quickstart](https://kafka.apache.org/quickstart) guide for installation instructions.
- .NET Core SDK installed on your machine. You can download it from the [.NET downloads](https://dotnet.microsoft.com/download) page.

## Steps

Follow these steps to create a simple publish and subscribe solution with Kafka and .NET:

1. **Set up Apache Kafka**

   Install and set up Apache Kafka by following the official [Kafka Quickstart](https://kafka.apache.org/quickstart) guide.

2. **Set up .NET Core**

   Install the .NET Core SDK on your machine by downloading it from the [.NET downloads](https://dotnet.microsoft.com/download) page.

3. **Create a new .NET Console Application**

   - Open your preferred IDE or use the command-line interface to create a new .NET Console Application project:
     ```bash
     dotnet new console -n KafkaPubSubSolution
     cd KafkaPubSubSolution
     ```

4. **Add the Confluent.Kafka NuGet package**

   - Add the Confluent.Kafka NuGet package to your project:
     ```bash
     dotnet add package Confluent.Kafka
     ```

5. **Write the publishing code**

   - Open the `Program.cs` file in your project and replace its contents with the following code:

     ```csharp
     using Confluent.Kafka;
     using System;

     namespace KafkaPubSubSolution
     {
         class Program
         {
             private const string KafkaBootstrapServers = "localhost:9092";
             private const string KafkaTopic = "mytopic";

             static void Main(string[] args)
             {
                 var config = new ProducerConfig
                 {
                     BootstrapServers = KafkaBootstrapServers
                 };

                 using (var producer = new ProducerBuilder<Null, string>(config).Build())
                 {
                     string input;
                     do
                     {
                         Console.WriteLine("Enter a message to publish (or 'exit' to quit):");
                         input = Console.ReadLine();

                         if (input.ToLower() != "exit")
                         {
                             var message = new Message<Null, string>
                             {
                                 Value = input
                             };

                             producer.Produce(KafkaTopic, message, deliveryReport =>
                             {
                                 if (deliveryReport.Error.Code != ErrorCode.NoError)
                                 {
                                     Console.WriteLine($"Message delivery failed: {deliveryReport.Error.Reason}");
                                 }
                                 else
                                 {
                                     Console.WriteLine($"Message delivered: {deliveryReport.Message.Value}");
                                 }
                             });
                         }

                     } while (input.ToLower() != "exit");

                     producer.Flush(TimeSpan.FromSeconds(10));
                 }
             }
         }
     }
     ```

6. **Write the subscribing code**

   - Add a new file named `Subscriber.cs` to your project and replace its contents with the following code:

     ```
    using Confluent.Kafka;
using System;

namespace KafkaPubSubSolution
{
    class Subscriber
    {
        private const string KafkaBootstrapServers = "localhost:9092";
        private const string KafkaTopic = "mytopic";

        public void ConsumeMessages()
        {
            var config = new ConsumerConfig
            {
                BootstrapServers = KafkaBootstrapServers,
                GroupId = "myconsumer",
                AutoOffsetReset = AutoOffsetReset.Earliest
            };

            using (var consumer = new ConsumerBuilder<Ignore, string>(config).Build())
            {
                consumer.Subscribe(KafkaTopic);

                try
                {
                    while (true)
                    {
                        var consumeResult = consumer.Consume();

                        if (consumeResult.Message != null)
                        {
                            Console.WriteLine($"Received message: {consumeResult.Message.Value}");
                        }
                    }
                }
                catch (ConsumeException ex)
                {
                    Console.WriteLine($"Error while consuming messages: {ex.Error.Reason}");
                }
                finally
                {
                    consumer.Close();
                }
            }
        }
    }
}
``` 
                 
This code sets up a Kafka consumer, subscribes to the specified topic, and continuously consumes messages from the topic. Any received messages will be displayed in the console. Remember to replace the placeholder topic name (mytopic) with the actual Kafka topic you want to subscribe to.

   
                 
- By following the steps outlined in the guide, you'll be able to create a fully functional solution that showcases the publish and subscribe pattern using Apache Kafka and .NET. This solution will enable you to publish messages to a Kafka topic and subscribe to that topic to consume the messages, demonstrating the powerful capabilities of Kafka and its integration with .NET.
