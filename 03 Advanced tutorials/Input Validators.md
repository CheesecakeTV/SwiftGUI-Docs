
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
|none|Don't validate at all (default)|
|all|Validate in any case|
|key|Validate when any key is pressed while focussing the element|

In theory, there are more modes, but they don't seem to work properly.
In practice, there should be no noticable difference between `all` and `key`.

Pass the mode as a string to the element:
```py
sg.Input(
    validate="all",
)
```

# Validators
Validators decide which input is allowed and which isn't.
If validation is enabled, an invalid input will not be possible - the user can't input invalid text.

Set the option `validatecommand` to the validator of your choice.

For example, let's restrict the length of the input text to 5:
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

Note that the validators never change the actual text/value.
`CanConvertTo(int)` will check if you can convert the text to an `int`, but won't do it itself.

Also, backend changes are not validated.
That means, you can set the text to something invalid from the code: `elem.value = ...`.

## Combining validators
Most (all) validators can be combined with and-/or-conditions.

For `AND` and `OR`, you'll need to use `&` and `|`.

For example, let's create an input that allows integers with a maximum of 4 digits:
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

# Creating custom validators
If you think, the implemented selection of validators is incomplete, feel free to create your own.

There are two base classes you may utilize:
`sg.Validate.BaseValidator` and `sg.Validate.BaseValidateValue`.

`BaseValidator` is less restrictive, but doesn't allow combinations (`&`, `|`, `-`).
Use `BaseValidateValue` if possible.

With `BaseValidateValue`, there are different methods to implement your own validation.
The change is valid, if ALL OF THESE METHODS return `True`.

## Full value check
The easiest check is for the full value.

Overwrite the method `is_valid_full_value` to return `True`, if the passed value is valid:
```py
import SwiftGUI as sg

sg.Themes.FourColors.Emerald()

class UppercaseOnly(sg.Validate.BaseValidateValue):

    def is_valid_full_value(self, val: str) -> bool:
        return val == val.upper()   # True, if the whole string is uppercase

layout = [
    [
        sg.Input(
            validate="all",
            validatecommand=UppercaseOnly(),    # Using the custom validator
        )
    ]
]

w = sg.Window(layout)
w.loop()
```

`val` gets passed the full element-value every time a validation occurs (at least after each change).

## Value-change checks
`is_valid_insertion(self, val: str, val_before: str, inserted_val: str) -> bool` is called when text is added to the value.

`def is_valid_deletion(self, val: str, val_before: str, deleted_val: str) -> bool` is called when text is removed from the value.

There are 3 parameters:
- `val`: The text AFTER the change being validated
- `val_before`: The text BEFORE the change
- `inserted_val`/`deleted_val`: The part of the text added/deleted

Keep in mind that you can add more than one character in a single change by pasting text it into the input.
That means, `inserted_val` can be longer than one character.

Let's forbid all spaces in an input:
```py
class NoSpaces(sg.Validate.BaseValidateValue):

    def is_valid_insertion(self, val: str, val_before: str, inserted_val: str) -> bool:
        return " " not in inserted_val
```

Let's forbid all deletions:
```py
class NoDeletions(sg.Validate.BaseValidateValue):

    def is_valid_deletion(self, val: str, val_before: str, deleted_val: str) -> bool:
        return False
```

Keep in mind that working with value-changes instead of full values has an interesting effect:
Text already in the input will not be validated again.

You can add invalid text from the backend and the user will still be able to insert text.

This is good, or annoying, depending on your application.

## BaseValidator
This is the more flexible base-class for validators, but won't be explained in this tutorial, since it would mostly describe tkinter topics.

To use this baseclass, first understand how validation works in tkinter:
https://anzeljg.github.io/rin2/book2/2405/docs/tkinter/entry-validation.html

Then, look at the implementation of the two base-classes.
I am sure you'll quickly understand the purpose of each attribute and method.

