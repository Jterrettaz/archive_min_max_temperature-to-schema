# archive_min_max_temperature-to-schema
Add the min and max temperature of each Davis Vantage Pro datalogger archive record into weewx database schema

In each Davis Vantage Pro datalogger archive record, there is 3 temperature values :
- the temperature measured at the time of archiving, or the mean temperature during the archive period, depending on the selected configuration.
- the lowest temperature measured since the last archive record.
- the highest temperature measured since the last archive record.

By default, using the vantage driver, the low and high temperature of each archive record is not recorded into the weewx database.

The configuration steps described below will allow Weewx to store the archive period min and max temperature into the weewx archive database.

## Configuration
 1. Add at the end of the file **extensions.py** located in the user directory (most often /usr/share/weewx/user or /home/weewx/bin/user) the following lines :
 ```python
   import schemas.wview
   import weewx.units
   schema_with_low_high_temp = schemas.wview.schema + [('lowOutTemp', 'REAL')] + [('highOutTemp', 'REAL')]
   weewx.units.obs_group_dict['lowOutTemp'] = 'group_temperature'
   weewx.units.obs_group_dict['highOutTemp'] = 'group_temperature'
 ```

 If you are using the new extended schema intruduced with Weewx V4.0.0 :
 ```python
   import schemas.wview
   import weewx.units
   schema_with_low_high_temp = schemas.wview_extended.schema + [('lowOutTemp', 'REAL')] + [('highOutTemp', 'REAL')]
   weewx.units.obs_group_dict['lowOutTemp'] = 'group_temperature'
   weewx.units.obs_group_dict['highOutTemp'] = 'group_temperature'
 ```
 2. Change weewx.conf to include the new database schema :
 ### with sqlite database :
 ```python
  [DataBindings]
     [[wx_binding]]
         # The database must match one of the sections in [Databases].
         # This is likely to be the only option you would want to change.
         database = archive_sqlite 
         # The name of the table within the database
         table_name = archive
         # The manager handles aggregation of data for historical summaries
         manager = weewx.wxmanager.WXDaySummaryManager
         # The schema defines the structure of the database.
         # It is *only* used when the database is created.
         #schema = schemas.wview.schema
         schema= user.extensions. schema_with_low_high_temp 
 ```
 
 ### with mysql database :
  ```python
  [DataBindings]
     [[wx_binding]]
         # The database must match one of the sections in [Databases].
         # This is likely to be the only option you would want to change.
         database = archive_mysql 
         # The name of the table within the database
         table_name = archive
         # The manager handles aggregation of data for historical summaries
         manager = weewx.wxmanager.WXDaySummaryManager
         # The schema defines the structure of the database.
         # It is *only* used when the database is created.
         #schema = schemas.wview.schema
         schema= user.extensions. schema_with_low_high_temp 
 ```
 
 3. Shutdown Weewx and update your database to bring in the new field.
       ```python
       wee_database weewx.conf --reconfigure
       ```
      This will create a new database (nominally, weewx.sdb_new if you are using SQLite, weewx_new if you are using MySQL) using the new schema and populate it with data from the old database.
   5. Shuffle the databases. Now arrange things so WeeWX can find the new database.
   **Make sure you know what you're doing at this point, you can potentially corrupt/lose your archive data.**
   You can either shuffle the databases around so the new database has the same name as the old database, or edit weewx.conf to use the new database name. To do the former:

 ###For SQLite:
  ```
    cd SQLITE_ROOT
    mv weewx.sdb_new weewx.sdb
  ```

  ###For Mysql: 
  ```
    mysql -u <username> --password=<mypassword>
    mysql> DROP DATABASE weewx;                             # Delete the old database
    mysql> CREATE DATABASE weewx;                           # Create a new one with the same name
    mysql> RENAME TABLE weewx_new.archive TO weewx.archive; # Rename to the nominal name
  ```
  
  4. Restart Weewx
  
  Two new fields, **lowOutTemp** and **highOutTemp** are now populated each time an archive record is added to the weewx archive database.
  These fields are accessible using Weewx tagd, for example :
  - Absolute lowest temperature of last hour :  **$hour.lowOutTemp.min**
  - Absolute highest temperature of the day : **$day.highOutTemp.max**

