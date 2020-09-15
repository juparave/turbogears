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

```
