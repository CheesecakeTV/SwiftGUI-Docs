
Written for SwiftGUI version 0.11.17 if not stated further.


# sg.Spacer
Alias: `S`

The spacer is a pseudo-invisible element that can be used to add some space between elements.

It has no default-event, no value (= `None`) and no outstanding methods.
It can have a key though.

# Options
- `width`: Width in pixels
- `height`: Height in pixels

# Tips
- Even though the element has no default event, you may attach events to it using `.bind_event`
- The width and height of the element can be updated using `.update`, so you could restructure your layout at runtime
