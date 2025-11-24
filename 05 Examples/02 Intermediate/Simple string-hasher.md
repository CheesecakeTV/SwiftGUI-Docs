
# Simple string-hasher
This example is an actually useful little program that can calculate hash-values.

The user can select which hash-algorithm to use and how to encode it:\
![](../../assets/images/2025-11-24-12-36-45.png)

# Demonstrated concepts
- Dynamic usage of functions (not SwiftGUI-native)
- Using multiple keys for the same functionality
- `sg.Combobox`
- Setting the focus
- Throwing an initial event
- Copying a value to clipboard

# Full code
Written in SwiftGUI version 0.10.18:
```py
import hashlib
import SwiftGUI as sg
import base64

sg.Themes.FourColors.RoyalBlue()

# Available functions
# Will be converted to a dict next
sha_algorithms = [
    hashlib.md5,
    hashlib.sha1,
    hashlib.sha1,
    hashlib.sha224,
    hashlib.sha256,
    hashlib.sha384,
    hashlib.sha512,
    hashlib.sha3_224,
    hashlib.sha3_256,
    hashlib.sha3_384,
    hashlib.sha3_512,
]

# dict where the key is the name and the value is the algorithm-function.
# Look at the following dict as a comparison.
sha_algorithms = {
    fct.__name__: fct for fct in sha_algorithms
}

# sha_algorithms looks like this, but I was too lazy to write every name manually.
enc_algorithms = {
    "Base 16": base64.b16encode,
    "Base 32": base64.b32encode,
    "Base 64": base64.b64encode,
    "Base 85": base64.b85encode,
}

def make_hash(text: str, algo_name: str, encoding_name: str) -> str:
    """
    Convert text to its hash-value and return it base_x encoded.

    :param text: Text to hash
    :param algo_name: hash-algorithm (key from the sha_algorithms-dict)
    :param encoding_name: encoding-algorithm (key from the enc_algorithms-dict)
    :return: Hash-value as a string
    """
    # You don't need to know how this works, this is an example for SwiftGUI after all.

    # Get the specified functions from the dictionaries
    sha_algo = sha_algorithms[algo_name]
    enc_algo = enc_algorithms[encoding_name]

    text = text.encode()    # Convert string to byte-string
    text = sha_algo(text).digest()  # Create the hash
    text = enc_algo(text)   # Encode to a base so it can be a readable string
    text = text.decode()    # Convert from byte-string back to string

    return text

layout = [
    [
        sg.Combobox(    # This lets the user select the hash-algorithm
            sha_algorithms.keys(),
            default_value= "openssl_sha3_256",
            key= "HashAlgo",
            default_event= True,
        ),
        sg.Combobox(    # encoding-algorithm
            enc_algorithms.keys(),
            default_value= "Base 64",
            key= "EncAlgo",
            default_event= True,
        )
    ],[
        sg.TextField(   # Entry for the input-text
            key= "Textfield",
            default_event= True,
        )
    ], [
        output := sg.TextField( # Output for the hash-value
            readonly= True,
            height= 2,  # 2 rows are enough space for the hash-value
        ),
    ],[
        sg.Button(
            "Copy to clipboard",
            key_function= lambda: sg.clipboard_copy(output.value)   # Copy the value of the output to clipboard
        )
    ]
]

w = sg.Window(layout, title= "String hasher")
w.throw_event("Textfield")  # The hasing should be executed once at start, so let's tell SwiftGUI the textfield changed
w["Textfield"].set_focus()  # Set the focus to this element so the user can directly start writing

for e, v in w:

    # Every key calls this, but using if, it's easier to expand
    if e in ["Textfield", "HashAlgo", "EncAlgo"]:
        output.value = make_hash(   # Calculate hash-value and write it to the output
            text= v["Textfield"],
            algo_name= v["HashAlgo"],
            encoding_name= v["EncAlgo"],
        )
```
