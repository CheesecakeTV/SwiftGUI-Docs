
# Popup Create Password
This example shows a popup from a program I am currently working on.

It asks the user to set and confirm a password:\
![](../../assets/images/2025-11-24-22-49-35.png)

It is especially interesting since the code is quite short for what it can do:
- The "confirm-field" is colored red when the contents of the fields don't match
- When checking "Show password", the actual letters in "password-field" show instead of `*****`
- Pressing enter in the "password-field" makes the focus jump to the "confirm-field"
- Pressing enter in the "confirm-field" does the same as clicking "Confirm"

# Demonstrated concepts
- Popups (blocking)
- Setting/changing the focus
  - Disabling takefocus for a better user-experience
- Intermediate use of key-functions
- Dynamic use of the event-loop
- Updating options

# Full code
Written in SwiftGUI version 0.10.17:
```py
import SwiftGUI as sg
from SwiftGUI import ValueDict

sg.Themes.FourColors.SinCity()

class create_password(sg.BasePopup, str):
    def __init__(self, title: str = "Create your password", **kwargs):

        layout = [
            [
                sg.T("Password:", width= 10),
                password := sg.In(
                    key= "PW",
                    default_event= True,
                    pass_char= "*", # Hidden characters
                ).bind_event(
                    sg.Event.KeyEnter,
                    key_function= lambda w:w["Confirm"].set_focus() # Jump to next input-element
                ),
                sg.Spacer(width=5),
                sg.Checkbox(
                    "Show password",
                    default_event= True,
                    key_function= lambda val: password.update(pass_char = "" if val else "*"),  # If the box is checked, reveal characters. Else hide them
                    takefocus= False,   # Pressing tab should ignore this element
                )
            ],[
                sg.T("Confirm:", width= 10),
                confirm := sg.In(
                    key= "Confirm",
                    default_event= True,
                    pass_char= "*", # Also hidden characters
                ).bind_event(
                    sg.Event.KeyEnter,  # Same as clicking "Confirm"
                    key= "Done",
                ),
                sg.T(expand=True)
            ],[
                sg.HSep(),
            ], [
                sg.Button(
                    "Confirm",
                    key= "Done",
                ),
                sg.Button(
                    "Cancel",
                    key_function= lambda :self.done()   # Call self.done() so the popup "returns" None
                )
            ]
        ]
        self.password = password    # Save these two for later
        self.confirm = confirm

        super().__init__(layout, title=title, **kwargs)
        password.set_focus()    # Start with the focus on the password-input-field

    def _event_loop(self, e: Hashable, v: sg.ValueDict):
        pws_match = self.password.value == self.confirm.value

        if e == "Done" and pws_match:
            self.done(self.password.value)  # "Return" the password

        # If any other key happened, check if the two input-values match
        if pws_match:
            # If they do, use the default background-color for the confirm-field
            self.confirm.update_to_default_value("background_color")
        else:
            # Else, set it to red
            self.confirm.update(background_color="#A52A2A")


# Using the popup
print("Created password:", create_password())
```


