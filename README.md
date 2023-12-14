# SkipNamesTypeInfoResolver

Emulates jsonIgnore on anonymous types (sort of), filters out also empty objects! -> object1:{}

```c#
internal class Program
{
	static readonly string[] Ignore = [  "Name1", "Name2", "Name3"  ];
	static bool ShouldSerialize(object? value)
	{
		if (value == null)
			return false;

		if (value is string s)
			return s.Length > 0;

		if (value is IList<object> list)
			return list.Count > 0;

		foreach (var property in value.GetType().GetProperties()) 
		{
			if (Array.IndexOf(Ignore, property.Name) >= 0)
				continue;

			if (ShouldSerialize(property.GetValue(value)))
				return true;
		}
		return false;
	}

	async static Task Main(string[] args)
	{
		var vm = DataEngine.GetObject();

		var json = JsonSerializer.Serialize(vm, new JsonSerializerOptions()
		{
			WriteIndented = true,
			TypeInfoResolver = new DefaultJsonTypeInfoResolver().WithAddedModifier(typeInfo =>
			{
				foreach (JsonPropertyInfo propertyInfo in typeInfo.Properties)
					propertyInfo.ShouldSerialize = (obj, value) =>
					{
						if (Array.IndexOf(Ignore, propertyInfo.Name) >= 0)
							return false;
						else
							return ShouldSerialize(value);
					};
			})
		});

		await File.WriteAllTextAsync(@"test1.json", json);
	}
}
```
