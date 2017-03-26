<p align="center">
<img src="https://github.com/chkr1011/MQTTnet/blob/master/Images/Logo_128x128.png?raw=true" width="128">
</p>

# MQTTnet
MQTTnet is a .NET library for MQTT based communication. It provides a MQTT client and a MQTT server.

## Supported frameworks

* .NET Framework 4.5.2+
* .NET Core 1.0+
* Universal Windows (UWP) 10.0.10240+

# MqttClient
## Example

```c#
var options = new MqttClientOptions
{
    Server = "localhost"
};

var client = new MqttClientFactory().CreateMqttClient(options);
client.ApplicationMessageReceived += (s, e) =>
{
    Console.WriteLine("### RECEIVED APPLICATION MESSAGE ###");
    Console.WriteLine($"+ Topic = {e.ApplicationMessage.Topic}");
    Console.WriteLine($"+ Payload = {Encoding.UTF8.GetString(e.ApplicationMessage.Payload)}");
    Console.WriteLine($"+ QoS = {e.ApplicationMessage.QualityOfServiceLevel}");
    Console.WriteLine($"+ Retain = {e.ApplicationMessage.Retain}");
    Console.WriteLine();
};

client.Connected += async (s, e) =>
{
    Console.WriteLine("### CONNECTED WITH SERVER ###");

    await client.SubscribeAsync(new List<TopicFilter>
    {
        new TopicFilter("#", MqttQualityOfServiceLevel.AtMostOnce)
    });

    Console.WriteLine("### SUBSCRIBED ###");
};

client.Disconnected += async (s, e) => 
{
    Console.WriteLine("### DISCONNECTED FROM SERVER ###");
    await Task.Delay(TimeSpan.FromSeconds(5));

    try
    {
        await client.ConnectAsync();
    }
    catch
    {
        Console.WriteLine("### RECONNECTING FAILED ###");
    }
};

try
{
    await client.ConnectAsync();
}
catch
{
    Console.WriteLine("### CONNECTING FAILED ###");
}

Console.WriteLine("### WAITING FOR APPLICATION MESSAGES ###");

while (true)
{
    Console.ReadLine();

    var applicationMessage = new MqttApplicationMessage(
        "A/B/C",
        Encoding.UTF8.GetBytes("Hello World"),
        MqttQualityOfServiceLevel.AtLeastOnce,
        false
    );

    await client.PublishAsync(applicationMessage);
}
```

# MqttServer

## Example 

```c#
var options = new MqttServerOptions
{
    ConnectionValidator = p =>
    {
        if (p.ClientId == "SpecialClient")
        {
            if (p.Username != "USER" || p.Password != "PASS")
            {
                return MqttConnectReturnCode.ConnectionRefusedBadUsernameOrPassword;
            }
        }

        return MqttConnectReturnCode.ConnectionAccepted;
    }
};

var mqttServer = new MqttServerFactory().CreateMqttServer(options);
mqttServer.Start();

Console.WriteLine("Press any key to exit.");
Console.ReadLine();

mqttServer.Stop();
```