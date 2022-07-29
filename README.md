Voir texte en français plus bas.
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
 
 If you are using the standard schema used before Weewx V4.0.0 :
 ```python
   import schemas.wview
   import weewx.units
   schema_with_low_high_temp = schemas.wview.schema + [('lowOutTemp', 'REAL')] + [('highOutTemp', 'REAL')]
   weewx.units.obs_group_dict['lowOutTemp'] = 'group_temperature'
   weewx.units.obs_group_dict['highOutTemp'] = 'group_temperature'
 ```

 If you are using the new extended schema introduced with Weewx V4.0.0 :
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
         schema= user.extensions.schema_with_low_high_temp 
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
         schema= user.extensions.schema_with_low_high_temp 
 ```
 
 3. Shutdown Weewx and update your database to bring in the new fields. (Weewx v4.5.0 or newer)
       ```python
       wee_database --add-column=lowOutTemp
       wee_database --add-column=highOutTemp
       ```
  
  4. Restart Weewx
  
  Two new fields, **lowOutTemp** and **highOutTemp** are now populated each time an archive record is added to the weewx archive database.
  These fields are accessible using Weewx tags, for example :
  - Absolute lowest temperature of last hour :  **$hour.lowOutTemp.min**
  - Absolute highest temperature of the day : **$day.highOutTemp.max**
  
  # archive_min_max_temperature-to-schema
Ajout dans la base de données de Weewx des températures mini et maxi de chaque période d'archivage (station Davis Vantage)

A chaque archivage dans le datalogger, la console de la VP2 enregistre 3 données de température :
- la température instantanée mesurée au moment de l'archivage, ou la température moyenne mesurée depuis le dernier archivage, selon la configuration choisie.
- la température minimale mesurée depuis l'archivage précédent
- la température maximale mesurée depuis l'archivage précédent.

Ce système permet de ne pas manquer les extrema de température, même si la période d'archivage est assez longue.

Les étapes ci-dessous permettront à Weewx d'enregistrer dans sa base de donnée les températures minimales et maximales de la période d'archivage, et de rendre ainsi accessible ces données via les tags de weewx.


## Configuration
 1. Ajouter à la fin du fichier **extensions.py** (situé dans le dossier "utilisateur" de weewx (le plus souvent /usr/share/weewx/user ou /home/weewx/bin/user)les lignes suivantes :
 
 Si vous utilisez le schéma standard utilisé par les versions antérieures à la V4.0.0 de Weewx
 ```python
   import schemas.wview
   import weewx.units
   schema_with_low_high_temp = schemas.wview.schema + [('lowOutTemp', 'REAL')] + [('highOutTemp', 'REAL')]
   weewx.units.obs_group_dict['lowOutTemp'] = 'group_temperature'
   weewx.units.obs_group_dict['highOutTemp'] = 'group_temperature'
 ```

 Si vous utilisez le nouveau schéma étendu introduit à partir de Weewwx V 4.0.0
 ```python
   import schemas.wview
   import weewx.units
   schema_with_low_high_temp = schemas.wview_extended.schema + [('lowOutTemp', 'REAL')] + [('highOutTemp', 'REAL')]
   weewx.units.obs_group_dict['lowOutTemp'] = 'group_temperature'
   weewx.units.obs_group_dict['highOutTemp'] = 'group_temperature'
 ```
 2. Modifier weewx.conf pour prendre en compte le nouveau schema :
 ### avec sqlite :
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
         schema= user.extensions.schema_with_low_high_temp 
 ```
 
 ### avec mysql :
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
         schema= user.extensions.schema_with_low_high_temp 
 ```
 
 3. Stopper Weewx et mettre à jour la base de donnée avec les nouveaux champs (Weewx v4.5.0 or plus récent). 
 Dans une fenêtre terminal:
       ```python
       wee_database --add-column=lowOutTemp
       wee_database --add-column=highOutTemp
       ```
  
  4. Redémarrer Weewx
  
  Les 2 nouveaux champs, **lowOutTemp** and **highOutTemp** sont maintenant ajoutés chaque fois qu'un enregistrement d'archive est ajouté dans la base de données d'archive de Weewx.
  
  Ces champs sont accessibles via les tags de Weewx, par exemple :
  - Température minimum absolue durant la dernière heure :  **$hour.lowOutTemp.min**
  - Température maximum absolue du jour : **$day.highOutTemp.max**

