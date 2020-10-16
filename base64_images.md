# Base64 Images

```python
import base64

with open("yourfile.ext", "rb") as image_file:
    encoded_string = base64.b64encode(image_file.read())

# convert bytes to string
decoded_string = encoded_string.decode("utf-8")
```

```python
def write_blob_cdn(self, base64_data, filename, filetype, path=""):
    """ Writes file to CDN as stream from base64 data """

    if '/' in filetype:
        filetype = filetype.split('/')[1]  # removing */ from mime_type

    hashed_filename = short_uuid_gen() + '.' + filetype

    file_path = "%s/%s" % (path, hashed_filename)
    log.debug('file_path: %s', file_path)

    try:
        base64_str = base64_data
        base64_str = base64_str[base64_str.find(",") + 1:]  # Strip unnecessary data 'data:jpeg' etc
        if sys.version_info[0] < 3:
            from cStringIO import StringIO
            f = StringIO()
            f.write(base64.decodestring(base64_str))
        else:
            f = io.BytesIO()
            f.write(base64.b64decode(base64_str))
        f.seek(0)
        obj_url = upload_object_via_stream(iter(f), file_path)

        log.info('Successful writing')
    except Exception as e:
        log.error('Error while writing blob')
        raise e

    return obj_url

```
