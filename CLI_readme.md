# Incorta CLI API

## Required Files : 
- incorta.py [ python api]
- requests   [ requests module required by the api ]

## How to use

```
python incorta.py <api_command> <arguments>....
```

### Supported api commands
#### Login
| Command | Description  | arguments  |
|---|:---|:---|
|login| login to server, returns session to be used by all other commands|\<url\> \<tenant\> \<username\> \<password\> |
||url| server url|
||tenant| tenant name|
||username| user name|
||password| password|
||verify| to verify the server certificate, connection failes in case of self signed certs if turned on [true/false] |


**Example** :
``` 
python incorta.py login https://xxx.yyy.zzz:443/incorta tenant_name user_name password
```


**Example with certificate verification off** :
``` 
python incorta.py login https://xxx.yyy.zzz:8443/incorta tenant_name user_name password false
```


Typicaly you need to save the session to use it in your script
```
session=`python incorta.py login http://xxx.yyy.zzz:0000/incorta tenant_name user_name password`
echo $session
python incorta.py do_another_command $session
```
* * *
#### Logout
| Command | Description  | arguments  |
|---|:---|:---|
|logout| logs out from the server|\<session\>|
||session| the current session to log out from|

**Example** :
``` 
python incorta.py logout $session
```
**You should logout when you finish, no need to exhaust the server with unused sessions**
* * *

#### List schemas
| Command | Description  | arguments |
|---|:---|:---|
|list_schemas| lists all the schemas owned by the logged in user|\<session\>|
||session| the current session to log out from|

**Example** :
``` 
python incorta.py list_schemas $session
```
* * *

#### Export schemas
| Command | Description  | arguments |
|---|:---|:---|
|export_schemas| exports schemas to an incorta package |\<session\> \<package_output_path\> \<schema\> \<schema2\> \<schema3\> ...  |
||session| the current session to log out from|
||package_output_path| path to save the exported package|
||schema| schema names separated by a space **names are case senstive** |


**Example** :
``` 
python incorta.py export_schemas $session demo_sales_and_hr.zip HR SALES
```
* * *

#### Import schemas
| Command | Description  | arguments |
|---|:---|:---|
|import_schemas| imports package to the current tenant |\<session\> \<package_path\> |
||session| the current session to log out from|
||package_path| path of the package to be imported|


**Example** :
``` 
python incorta.py import_schemas $session demo_sales_and_hr.zip
```
* * *

#### Load Schemas
| Command | Description  | arguments |
|---|:---|:---|
|load_schema| loads the schema |\<session\> \<schema name\> [\<table name\>] |
||session| the current session to log out from|
||schema name| schema to be loaded **names are case senstive**|
||table name| table name within the schema to be loaded **names are case senstive**, |
||| if the table name wasn't provided the schema will be fully loaded |

**Example** [full load] :
``` 
python incorta.py load_schema $session SALES
```

**Example** [table load] :
``` 
python incorta.py load_schema $session SALES Customers
```
* * *

#### Load Schemas incrementally 
| Command | Description  | arguments |
|---|:---|:---|
| load_schema_incremental| loads the schema |\<session\> \<schema name\> |
||session| the current session to log out from|
||schema name| schema to be loaded **names are case senstive**|

**Example**  :
``` 
python incorta.py load_schema_from_snapshot $session SALES
```
* * *

#### Load Schemas from snapshots
| Command | Description  | arguments |
|---|:---|:---|
| load_schema_from_snapshot| loads the schema |\<session\> \<schema name\> |
||session| the current session to log out from|
||schema name| schema to be loaded **names are case senstive**|

**Example**  :
``` 
python incorta.py load_schema_from_snapshot $session SALES
```
* * *

### Full sample Schema [Export / Import]
Export schemas from one tenant and import them to another tenant[or server].

```
incorta_cmd="python incorta.py"
session=`$incorta_cmd login http://server1/incorta tenant1 user_name password`
$incorta_cmd export_schemas $session export.zip schema1 schema2 schema3
$incorta_cmd logout $session


session=`$incorta_cmd login http://server2/incorta tenant2 user_name password`
$incorta_cmd import_schemas $session export.zip
$incorta_cmd logout $session
```
***


#### Export Dashboards
| Command | Description  | arguments |
|---|:---|:---|
|export_dashboards| exports schemas to an incorta package |\<session\> \<package_output_path\>  "/path/to/dashboard/" "/another/path/*" ...  |
||session| the current session to log out from|
||package_output_path| path to save the exported package|
||dashboard path| dashboard paths seprated by space, surround the pathes with qoutes in case the name has spaces in it **names are case senstive** |


**Example** :
``` 
python incorta.py export_dashboards $session "/folder/subfolder/*" "/folder/dashboard" "dashboard_on_root_folder"
```
* * *

#### Import Dashboards
| Command | Description  | arguments |
|---|:---|:---|
|import_dashboards| imports package to the current tenant |\<session\> \<package_path\> |
||session| the current session to log out from|
||package_path| path of the package to be imported|
||overwrite| overwrite existing items [true / false] defaults to false|



**Example** :
``` 
python incorta.py import_dashboards $session demo_dashboards.zip
```

**Example with overwrite** :
``` 
python incorta.py import_dashboards $session demo_dashboards.zip true
```
* * *

#### Synchronize directory 
| Command | Description  | arguments |
|---|:---|:---|
|sync_directory| synchronize the system users, groups and their associations using the provided input |\<session\> \<archive_path\> |
||session| the current session to use for executing the actions. Should be a super user session.|
||archive_path| Contains three files (groups.csv, users.csv and user-groups.csv). Sample of each file is provided below|
||full_sync| flush all existing users/group associations before import. Users and groups are not removed automatically from the system. They are just reported to be deleted.|

**Sample groups.csv content**

|Name   | Description   |
|-------|---------------|
|group1 | Sample group 1|
|group2 | sample group 2|

**Sample users.csv content**

|login name| email| display name| language| country| timezone| calendar
|---|:---|:---|:----|:---|:---|:---|:---|
|user_1| user1@incorta.com| Sample User 1||||
|user_2| user2@incorta.com| Sample User 2||||
|user_3| user3@incorta.com| Sample User 3|||||

**Sample user-groups.csv content**

|group name| user login name|
|-------|---------------|
|group1 | user_1|
|group1 | user_2|
|group2 | user_3|

**_NOTE_**

```
login name and email are mandatory and should be unique. 
If the login name already exists in the tenant, the existing one will be updated.
If the email, already exists, the record will be rejected.

**Only super user is allowed to do this action**
```

**Example** :
``` 
python incorta.py sync_directory $session directory.zip
```

**Example with full sync** :
``` 
python incorta.py sync_directory $session directory.zip true
```
* * *


#### Synchronize directory with database
| Command | Description  | arguments |
|---|:---|:---|
|sync_directory_with_db| synchronize the system users, groups and their associations with data from a database |\<session\> \<connection_string\> \<class_name\> \<user\> \<password\> \<groups_query\> \<users_query\> \<groups_users_query\> \<full_sync\> |
||session| the current session to use for executing the actions. Should be a super user session.|
||connection_string| JDBC connection string for the database. For example in MySql, it will be something like: jdbc:mysql://localhost/sec_db|
||class_name| JDBC driver class name. For example in MySql, it will be something like: com.mysql.jdbc.Driver|
||user| database user name|
||password| database user password|
||groups_query| SQL query to get the groups to be imported|
||users_query| SQL query to get the users|
||groups_users_query| SQL query to get the groups/users association|
||full_sync| flush all existing users/group associations before import. Users and groups are not removed automatically from the system. They are just reported to be deleted.|

**Example to import from EBS** :
``` 
python incorta.py sync_directory_with_db $session "jdbc:oracle:thin:@<host>:1521:<SID>" "oracle.jdbc.OracleDriver" "apps" "apps" "select distinct fr.responsibility_name from fnd_responsibility_tl fr" "select fu.user_name, 'someone' || rownum || '@somewhere.com' email, he.full_name, '' language, '' country, '' timezone, '' calendar from fnd_user fu left outer join hr_employees he on fu.employee_id = he.employee_id " "select c.RESPONSIBILITY_NAME, a.USER_NAME from apps.FND_USER a, apps.FND_USER_RESP_GROUPS b, apps.FND_RESPONSIBILITY_TL c where a.USER_ID=b.USER_ID and b.RESPONSIBILITY_ID=c.RESPONSIBILITY_ID  and (sysdate between b.START_DATE and b.END_DATE OR b.END_DATE IS NULL)"
```
```
Please note that the above queries are just examples. In real life you should use real emails and email must be unique to be able to import the user. If email is not unique, users will be rejected.
```
* * *


