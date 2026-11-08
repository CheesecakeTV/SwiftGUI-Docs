
Written for SwiftGUI version 0.12.0.

# Input validation
In this case, input validation means preventing the user from making "forbidden" inputs to an `sg.Input`.

In SwiftGUI, this kind of input validation is implemented using "Validators".
Validators are very easy to use and you can create your own ones.

Everything in this tutorial works for `sg.Combobox` too.

# Activate validation
To make an input validate, you need to specify when the validation should happen.

There are a couple of different options:
|Mode|Description|
|-|-|
|all|Validate in any case|
|key|Validate when any key is pressed while focussing the element|

The default value is `none`.

Pass the mode as a string to the element:
```py
sg.Input(
    validate="all",
)
```

# Validators
Set the option `validatecommand` to the validator that validates however you like.

For example, let's restrict the length of the input to 5:
```py
sg.Input(
    validate="key",
    validatecommand=sg.Validate.MaxLength(5),
)
```

These validators are implemented in SwiftGUI version 0.12.0:
- `CanConvertTo`: The value must be convertible to the specified type. You may pass multiple types, meaning the value must be convertible to AT LEAST ONE of the types
- `MaxLength`: The value cannot exceed the specified amount of characters
- `ForbidDeletions`: The user can't delete characters
- `ForbidInsertions`: The user can't add (insert) characters
- `WhitelistCharacters`: Only the specified characters are allowed
- `FullValueFunction`: Apply your own function to check if the value is allowed
- `ReFullMatch`: Only allowed, if the provided regex-pattern fully matches

## Combining validators
Usual validators can be combined with and/or conditions.

Instead of the usual `AND` and `OR`, you'll need to use `&` and `|`.

For example, let's create an input that allows an integer with a maximum of 4 digits:
```py
sg.Input(
    validate="key",
    validatecommand= sg.Validate.CanConvertTo(int) & sg.Validate.MaxLength(4),
)
```

Or, let's create a field that allows any inputs up to 4 characters, but up to 8 characters for integers:
```py
sg.Input(
    validate="key",
    validatecommand= sg.Validate.MaxLength(4) | (sg.Validate.CanConvertTo(int) & sg.Validate.MaxLength(8)),
)
```

## Negating validators
You can make validators act the opposite by placing a minus (`-`) before it.

For example, let's create a field that accepts all characters except for a and b:
```py
sg.Input(
    validate="key",
    validatecommand= - sg.Validate.WhitelistCharacters("aAbB"),
)
```

## Validation-function
You can let your own function decide if a value is valid or not.

To do so, define a function that takes a single string as an argument and returns if the input is valid as a bool:
```py
def is_valid(val: str) -> bool:
    """Only allow capital characters"""
    return val.upper() == val.upper()
```
Then pass that function to the `FullValueFunction` validator:
```py
sg.Input(
    validate="key",
    validatecommand= sg.Validate.FullValueFunction(is_valid),
)
```
