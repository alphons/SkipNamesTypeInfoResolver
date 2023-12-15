# IgnoringJsonSerializer

Emulates jsonIgnore on anonymous types (sort of), filters out also empty objects! -> object1:{}

## IgnoringJsonSerializer
```c#

using System.Collections;
using System.Text.Json.Serialization.Metadata;

namespace System.Text.Json;

public class IgnoringJsonSerializer
{
	private static string[] Ignoring = [];
	private static bool ShouldSerialize(object? value)
	{
		if (value == null)
			return false;

		if (value is string s)
			return s.Length > 0;

		if (value is IList list)
			return list.Count > 0;

		foreach (var property in value.GetType().GetProperties())
		{
			if (Array.IndexOf(Ignoring, property.Name) >= 0)
				continue;

			if (ShouldSerialize(property.GetValue(value)))
				return true;
		}
		return false;
	}

	public static string Serialize(object o,
		JsonSerializerOptions? options = null,
		string[]? Ignore = null)
	{
		Ignoring = Ignore ?? ([]);
		options ??= new JsonSerializerOptions();

		options.TypeInfoResolver = new DefaultJsonTypeInfoResolver()
			.WithAddedModifier(typeInfo =>
			{
				foreach (JsonPropertyInfo propertyInfo in typeInfo.Properties)
					propertyInfo.ShouldSerialize = (obj, value) =>
					{
						if (Array.IndexOf(Ignoring, propertyInfo.Name) >= 0)
							return false;
						else
							return ShouldSerialize(value);
					};
			});
		return JsonSerializer.Serialize(o, options);
	}
}

```

## Test program

```c#
using System.Text.Json;
using VMTux.VMTuxEmulator.DataModel.Form;

namespace ConsoleApp1;

internal class Program
{
	async static Task Main(string[] args)
	{
		var vm = DataEngine.GetVm();

		var json0 = JsonSerializer.Serialize(vm, new JsonSerializerOptions() {  WriteIndented = true });
		var json1 = IgnoringJsonSerializer.Serialize(vm, new JsonSerializerOptions() { WriteIndented = true }, 
			["Icon", "Title", "Usage", "Type", "Options", "Default"]);

		await File.WriteAllTextAsync("json0.json", json0);
		await File.WriteAllTextAsync("json1.json", json1);
	}
}

```
