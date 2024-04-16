
Here is what i can read from the binary:
- First, it calls "parse", that seems to load file data in memory, and check if it is well formed
- Then it calls "check", that verifies if the content is matching security checks or something.

From what i understand from "parse":
- The file must begin with `----BEGIN WHITE LICENSE----\n`
- Last 28 chars must be: `-----END WHITE LICENSE-----\n`
- Inner content must be in base 64

The content of the given file is:
```
Name: Walter White Junior
Serial: 1d117c5a-297d-4ce6-9186-d4b84fb7f230
Type: 1
```

Then:
- It checks if there is a "name:", and if there is an \n later.
- Then extract the serial id ?
- Then check if there is a type
