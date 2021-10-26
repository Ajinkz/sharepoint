# sharepoint
office 365 - sharepoint - python


```python

import os
#from config import config
from shareplum import Site
from shareplum import Office365
from shareplum.site import Version


# get data from configuration
username = '' # user mail id
password = ''
basepath = '' # 'https://xxxxxxx.sharepoint.com/'
sitename = '' # 'https://xxxxxxx.sharepoint.com/sites/xxxxxx'
dirname = ''  # directory where file is suppose to upload

authcookie = Office365(basepath, username=username, password=password).GetCookies()

site = Site(sitename,version=Version.v365, authcookie=authcookie)

spfolder = site.Folder('Shared Documents/'+ dirname)

# filename along with its path
filepath = "exported-data.csv"

with open(filepath, 'rb') as file_input:
    try: 
        spfolder.upload_file(file_input, filepath)
        print("file uploaded")
    except Exception as err: 
        print("Some error occurred: " + str(err))

```
