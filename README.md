## Sharepoint file upload


### [Upload file using Graph API](graphapi_upload.py )

#### Prequisite
 - Office365 account crendentials
 - Azure App with credentials and required scopes


Site id of required sharepoint site
```
GET https://graph.microsoft.com/v1.0/sites/organization.sharepoint.com:/sites/sharedlib
Site id can be found in return object inside id string (among 3 values 2nd value is site id)
id: site, site_id, web.id
```

Site's drive_id
```
GET https://graph.microsoft.com/v1.0/sites/{site_id}/drive
id: drive_id
```
Upload file in root folder i.e. https://organization.sharepoint.com/sites/site-name/Shared%20Documents/Forms/AllItems.aspx

```
PUT https://graph.microsoft.com/v1.0/drives/{drive_id}/root:/{file_url}:/content
```

Upload file in specific folder i.e /folderA/folderB
```
PUT https://graph.microsoft.com/v1.0/drives/{drive_id}/root:/{directory-relative-path}/{file_url}:/content
```
Using Graph API
```python
import adal
import json
import requests

tenant = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
app_id = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
app_password = 'nlZ7Q~n2pju.sPGrFwl7BxG_dCGS6VSlFXx9D'
userAccount = 'user@organization.onmicrosoft.com'
SHAREPOINT_HOST_NAME = 'organization.sharepoint.com'
resource_URL ='https://graph.microsoft.com'
authority_url = 'https://login.microsoftonline.com/%s'%(tenant)

relative_site_path = "/sites/sharedlib" # site name
dirname = "csvdump"
file_url = "test-2021-11-12.csv"

# Authentication
context = adal.AuthenticationContext(authority_url)

token = context.acquire_token_with_client_credentials(
    resource_URL,
    app_id,
    app_password)

request_headers = {'Authorization': 'bearer %s'%(token['accessToken'])}

# Get Site id ------------
# GET https://graph.microsoft.com/v1.0/sites/organization.sharepoint.com:/sites/sharedlib
site_api = f"{ resource_URL }/v1.0/sites/organization.sharepoint.com:{relative_site_path}"
site_api_res = requests.get(site_api, headers=request_headers)
if site_api_res.status_code == 200:
    print("succeed\n",site_api_res.json())
    my_site_id = site_api_res.json()['id'].split(',')[1]
else:
    print("failed",site_api_res.json())
    
# site's speceific drive 
# GET https://graph.microsoft.com/v1.0/sites/53a06a67-2386-4161-adfc-9c575e9096aa/drive
    
drive_api = f"{ resource_URL }/v1.0/sites/{my_site_id}/drive"
drive_api_res = requests.get(drive_api, headers=request_headers)
if drive_api_res.status_code == 200:
    print("succeed\n",drive_api_res.json())
    drive_id = drive_api_res.json()['id']
else:
    print("failed",drive_api_res.json())

# upload file
# PUT https://graph.microsoft.com/v1.0/drives/drive_id/root:/dirname/file_url:/content'

file_upload_api = f"{ resource_URL }/v1.0/drives/{drive_id}/root:/{dirname}/{file_url}:/content"
#upload_api = resource_URL+ "/drives/"+drive_id+"/root:/csvdump/"+filename+":/content"

file_upload_res = requests.put(file_upload_api,
                   headers={"Authorization": "Bearer " + token['accessToken'],
                     "Content-Type": "application/json"},
                   data=open(file_url, 'rb'))

if file_upload_res.status_code == 201:
    print("succeed\n",file_upload_res.json())
else:
    print("failed",file_upload_res.json())   
    

```


### [Upload file using Shareplum library](shareplum_upload.py)


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

References

- https://docs.microsoft.com/en-us/sharepoint/dev/sp-add-ins/working-with-folders-and-files-with-rest
- https://docs.microsoft.com/en-us/onedrive/developer/rest-api/resources/site?view=odsp-graph-online
- https://stackoverflow.com/questions/45406451/how-can-i-get-the-siteid-of-the-current-site-with-microsoft-graph-api
- 
