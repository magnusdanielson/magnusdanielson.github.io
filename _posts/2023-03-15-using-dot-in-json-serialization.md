---
layout: post
title:  "Keeping odata.nextLink property in a c# class"
author: magnus
categories: [ rest, api, sharepoint, json ]
image: assets/images/17.jpg
---

The SharePoint REST API produces properties with a dot (.) in their names. That caused me some troubles when I wanted to deserialize it to a C# class since I wanted to keep track of the `odata.nextLink`. It is possible to do anyway. Assuming you have a JSON string that contains a property with a dot in its name, you can use the Newtonsoft.Json library to deserialize it in C#. Here's an example code snippet:

```c
using Newtonsoft.Json;

public class Person
{
    public string Firstname { get; set; }
    public string Lastname { get; set; }
    public int Age { get; set; }
    [JsonProperty("address.city")] // Use JsonProperty attribute to specify the actual JSON property name
    public string City { get; set; }
}

// JSON string
string json = "{\"Firstname\":\"John\",\"Lastname\":\"Doe\",\"Age\":30,\"address.city\":\"New York\"}";

// Deserialize JSON string to Person object
Person person = JsonConvert.DeserializeObject<Person>(json);

// Access properties
Console.WriteLine(person.Firstname); // Output: John
Console.WriteLine(person.Lastname); // Output: Doe
Console.WriteLine(person.Age); // Output: 30
Console.WriteLine(person.City); // Output: New York
```

In the Person class, we use the [JsonProperty] attribute to specify the actual JSON property name for the City property. When deserializing the JSON string using JsonConvert.DeserializeObject<T>(), the library will map the properties based on the property names in the class and the JSON object, including the one with a dot in its name.