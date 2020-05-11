# archive_min_max_temperature-to-schema
Add the min and max temperature of each Davis Vantage Pro datalogger archive record into weewx database schema

In each Davis Vantage Pro datalogger archive record, there is 3 temperature values :
- the temperature measured at the time of archiving, or the mean temperature during the archive period, depending on the selected configuration.
- the lowest temperature measured since the last archive record.
- the highest temperature measured since the last archive record.

By default, using the vantage driver, the low and high temperature of each archive record is not recorded in the weewx database.

The configuration steps described below will allow Weewx to store the archive period min and max temperature into the weewx archive database.


