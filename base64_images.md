# Base64 Images

```pythoon
import base64

with open("yourfile.ext", "rb") as image_file:
    encoded_string = base64.b64encode(image_file.read())

# convert bytes to string
decoded_string = encoded_string.decode("utf-8")
```
