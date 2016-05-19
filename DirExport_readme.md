# Synchronize directory 

This feature can be used to synchronize system users, groups, along with their assignments using a zip file containing 3 CSV files (users.csv, groups.csv, user-groups.csv).

To use this endpoint, the user has to have a super-user privilege.
It returns the status of rows; which rows succeeded and which failed.

This endpoint can be accessed using Incorta’s CLI API (Please refer to its documentation for more details on how to use).

Here is a sample on how to use the “Synchronize Directory” feature through the CLI API:

```session=python incorta.py login <server-host> <tenant-name> <user-name> <password>```

```python incorta.py sync_directory $session <archive_path> <full_sync>```

| Parameters | Description |
|---|:---|
|session| the current session to use for executing the actions. Should be a super user session.|
|archive_path| Contains three files (groups.csv, users.csv and user-groups.csv). Sample of each file is provided below|
|full_sync (optional)| The default value for this parameter is set to “false”. If set to “true”: The assignments of all the users and groups would get flushed before the import. Existing users and groups would be updated regardless of this option value.|

**“groups.csv” file content**

| Parameter | Notes |
|---|:---|
| Name | This is the group name, and it is a  mandatory column. |
| Description | This is the group description, and it is a mandatory column that is nullable (value can be blank). |
| Type | Group Type. Column is optional. Possible values (case insensitive): **1 internal** [Default]: Internal incorta group. **2- sso**: Behaves the same like internal groups. It only indicates that the group is imported from an SSO . **3- ldap** : the group is maintained by an ldap directory server. Please review ldap support documentation to get more details about LDAP integration with incorta security. |
| ExternalId | This column is optional. Required only if the type is ldap when it holds the value of the “Group Distinguished Name”. |

**Sample**

|Name   | Description   |   Type   | externalId |
|-------|---------------|----------|------------|
|group1 | Sample group 1| INETRNAL |            |
|group2 | sample group 2| SSO      |            |

**“users.csv” file content**

| Parameter | Notes |
|---|:---|
| Login Name | Column is mandatory and unique |
| Email | Column is mandatory and unique |
| Display Name | Column is mandatory |
| Language | Column is mandatory but it is nullable (value can be blank) |
| Country | Column is mandatory but it is nullable (value can be blank) |
| Timezone | Column is mandatory but it is nullable (value can be blank) |
| Calendar | Column is mandatory but it is nullable (value can be blank) |
| Type | User Type : Column is optional. Possible values (case insensitive): **1- internal** [Default]: Internal incorta user. Incorta will be handling password management (store, encrypt, change, reset, forgot). **2- sso** : user will be authenticated by an SSO gateway before reaching incorta server. No password will be managed by incorta. SSO gateway must send incorta a user identity that matches the login name imported here. **3- ldap** : the user will be authenticated against an ldap directory server reachable by incorta server. Please review ldap support documentation to get more details about LDAP integration with incorta security. **Note**: in case of internal, user's password will be set to the same value of the login name, and the user will be asked to change it the first time he will login to the system.
| External Id | Column is optional. Required only if the type is **ldap** when it holds the value of the “User Distinguished Name”. |

**Sample**

|login name| email| display name| language| country| timezone| calendar| type| externalId|
|---|:---|:---|:----|:---|:---|:---|:---|:---|
|user_1| user1@incorta.com| Sample User 1|||||INTERNAL|
|user_2| user2@incorta.com| Sample User 2|||||SSO|
|user_3| user3@incorta.com| Sample User 3|||||SSO||

**“user-groups.csv” file content**

| Parameter | Notes |
|---|:---|
| Group Name | Column is mandatory. |
| Login Name | Column is mandatory. |

**Sample**

|group name| user login name|
|-------|---------------|
|group1 | user_1|
|group1 | user_2|
|group2 | user_3|

**_NOTE_**

```
If you have a field which includes a comma, the whole field should be encapsulated with double quotes.
Your csv files should not contain unnecessary spaces.
The order of the columns is very important.
```


# How to export with dirExport tool
  This tool is only a helper tool to prepare the input files for **syncDirectory** API. It helps you export your data (users, groups, their relations) from database or ldap into csv files encapsulated in a zip file.

#### Export from database
- Windows systems:

```dirExport -db db-config.properties [--debug]```
- Linux based systems:

```./dirExport.sh -db db-config.properties [--debug]```

  -- [--debug]: a flag to enable the debug mode.

###### db-config.properties has to contain the following properties (attached a sample file):
- connectionString: JDBC connection string for the database. For example in MySql, it will be something like: jdbc:mysql://localhost/sec_db
- driverClass: JDBC driver class name. For example in MySql, it will be something like: com.mysql.jdbc.Driver
- user: database user name
- password: database user password
- groupsQuery: SQL query to get the groups to be imported
  (original columns in the source table must be aliased using those labels [GROUPNAME], [DESCRIPTION])
- usersQuery: SQL query to get the users
  (original columns in the source table must be aliased using those labels [LOGINNAME], [EMAIL], [NAME], [LANGUAGE], [COUNTRY], [TIMEZONE], [CALENDAR])
- assignmentsQuery: SQL query to get the groups/users association
  (original columns in the source table must be aliased using those labels [LOGINNAME], [GROUPNAME])
- user.type: optional and can be one of [internal, sso] (default: internal).

###### Note:
- If you use a database not supported by the server, you will have to put its driverClass beside the dirExport.jar, OR you can edit the script file to include the path of the driverClass in the classpath.
- All columns are mandatory in groupsQuery and assignmentsQuery. In usersQuery, the following columns are optional and the column order is not necessary: ([LANGUAGE], [COUNTRY], [TIMEZONE], [CALENDAR])

#### Export from LDAP server
- Windows systems:

```dirExport -ldap ldap-config.properties [--debug]```
- Linux based systems

```./dirExport.sh -ldap ldap-config.properties [--debug]```

  -- [--debug]: a flag to enable the debug mode.
  
###### ldap-config.properties has to contain the following properties (attached a sample file):
- ldap.base.provider.url: Address of directory server for the backend to communicate with
- ldap.base.dn: Distinguished Name to connect with while accessing the server
- ldap.user.dn: Distinguished Name of a user in the ldap to authenticate with
- ldap.user.dn.password: Password for the authentication user
- ldap.user.mapping.login: the attribute that will map the login name of Incorta User
- ldap.user.mapping.name: the attribute that will map the name of Incorta User
- ldap.user.mapping.mail: the attribute that will map the mail of Incorta User
- ldap.group.mapping.name: the attribute that will map the name of Incorta group
- ldap.group.mapping.member: the attribute that will map the users in ldap group
- ldap.user.search.filter: How to look for users (fiter)
- ldap.group.search.filter: How to look for groups (fiter)
- user.type: optional and can be one of [internal, sso, ldap] (default: ldap).


# Synchronizing External Users Automatically with Incorta Analytics

You may automatically synchronize users with Incorta Analytics without having to use the “Synchronize Directory” or the “dirExport” tools separately. First, you would need the following files:

- sync_directory.py [python api]
- requests [requests module required by the api]

You would also need to identify the original location of the users to be imported and use the appropriate script from the following:

- **sync_directory_with_db.sh**: Use this script if the users are to be imported from a database. The script calls the directory export tool “dirExport”, then calls the synchronize directory endpoint with the output. (Note that the “db-config.properties” file has to be in the same folder as the “sync_directory.py” file.)
- **sync_directory_with_ldap.sh**: Use this script if the users are to be imported from an LDAP Server. The script calls the directory export tool “dirExport”, then calls the synchronize directory endpoint with the output. (Note that the “ldap-config.properties” file has to be in the same folder as the “sync_directory.py” file.)

Two arguments are expected with the above scripts:

- session: the current active session to use it to call the endpoint.
- full_sync: the full_sync argument to pass to the directory export tool, this argument is optional with default false.

Example:
- python sync_directory.py sync_directory_with_db $session [full_sync]
- python sync_directory.py sync_directory_with_ldap $session [full_sync]
