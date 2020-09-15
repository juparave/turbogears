# libcloud for Rackspace Cloudfiles

[Apache Libcloud](http://libcloud.apache.org/)

Install

    $ pip install apache-libcloud

```python
import libcloud
from libcloud.compute.types import Provider
from libcloud.compute.providers import get_driver

api_key = "xxxx2f5658bc65df8cb3a39f09a7bfe7"
username = "username"

# cloudfiles is the driver for RackSpace
cls = libcloud.get_driver(libcloud.DriverType.STORAGE, libcloud.DriverType.STORAGE.CLOUDFILES)
# all files are stored in the nearest location Dallas Forth Worth
driver = cls(username, api_key, region='dfw')
# create container
container = driver.create_container("test")
# if container already created, get the container
container = driver.get_container("test")
```

## Upload Object

[Examples](https://libcloud.readthedocs.io/en/stable/storage/examples.html)

```python
file_path = '/Users/pablito/Downloads/119136872_4470785782964095_7197786911271636876_n.jpg'
file_name = '119136872_4470785782964095_7197786911271636876_n.jpg'

# upload_object(file_path, container, object_name, extra=None, verify_hash=True, headers=None)
obj = driver.upload_object(file_path, container, file_name)
```

Adding optional extra values

```python
extra = {'meta_data': {'owner': 'myuser', 'created': '2020-09-15'}}
obj = driver.upload_object(file_path, container, file_name)
```

Uploading using streams

```python
with open(file_path, 'rb') as iterator:
    obj = driver.upload_object_via_stream(iterator=iterator,
                                          container=container,
                                          object_name='file_name',
                                          extra=extra)
```


