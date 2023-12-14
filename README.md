# SkipNamesTypeInfoResolver

Emulates jsonigonore on anonymous types (sort of)

```c#
var forbidden = new[] { "title", "usage", "type", "icon", "options", "default" };
var json = JsonSerializer.Serialize(vm, new JsonSerializerOptions()
{
	DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
	WriteIndented = true,
	TypeInfoResolver = new DefaultJsonTypeInfoResolver().WithAddedModifier(typeInfo =>
	{
		foreach (JsonPropertyInfo propertyInfo in typeInfo.Properties)
		{
			if (Array.IndexOf(forbidden, propertyInfo.Name) >= 0)
			{
				propertyInfo.ShouldSerialize = (obj, val) => false;
			}
			else
			{
				propertyInfo.ShouldSerialize = (obj, val) =>
				{
					if(val is IList<object>)
					{
						if ((val as IList<object>).Count == 0)
							return false;
						else
							return true;
					}
					if(val is string)
					{
						if ((val as string).Length == 0)
							return false;
						else
							return true;
					}

					// get all the Anonymous types
								
					var t = val.GetType();
					var tn = t.Name;

					if (!tn.Contains("Anonymous"))
						throw new Exception("Oops");

					var ps = t.GetProperties();

					foreach(var p in ps)
					{
						if (Array.IndexOf(forbidden, p.Name) < 0)
							return true;
					}
					return false;
				};
			}
		}
	})
});

```
