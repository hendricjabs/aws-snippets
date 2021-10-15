# Decode DynamoDB JSON

```py
def dynamodb_decode(obj):
  # https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.DataTypes.html
  dynamodb_datatypes = ["S", "N", "M", "BOOL", "L", "B", "SS", "NS", "BS"]
  for datatype in dynamodb_datatypes:
    if isinstance(obj, dict):
      if datatype in obj:
        return obj[datatype]
  return obj
  
import json
sample_object = ...the dynamodb object as string...
decoded_object = json.loads(sample_object, object_hook=dynamodb_decode)
```
