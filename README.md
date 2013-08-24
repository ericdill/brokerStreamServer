A python client for retrieving data from an EPICS Channel Archiver.

To get started just import the `Archiver` class and specify the address of your Channel Archiver server:

```python
from channelarchiver import Archiver
archiver = Archiver('http://cr01arc01/cgi-bin/ArchiveDataServer.cgi')
```

You then fetch data with the `archiver.get()` method:

```python
>>> data = archiver.get('SR00IE01:INJECTION_EFFICIENCY_MONITOR', '2013-08-11', '2013-08-12')
>>> print data
               time        value     status      severity
2013-08-11 00:00:02   96.9351518   NO_ALARM      NO_ALARM
2013-08-11 00:04:20   94.5171233   NO_ALARM      NO_ALARM
2013-08-11 00:08:38   85.0604361  LOW_ALARM         MINOR
...
>>> data.values
[96.935, 94.517, ..., 97.253]
```

The returned `ChannelData` object has the following fields:

* `channel`: The channel name.
* `times`: A list of datetimes.
* `values`: A list of the channel's values corresponding to `times`.
* `severities` and `statuses`: Diagnostic information about the channel state for each time.
* `units`: The units of `values`.
* `states`: String values for enum type channels.
* `data_type`: Whether the channel values are string, enum, int or double (see `codes.data_type`).
* `elements`: The number of elements in an array type channel.
* `display_limits`, `warn_limits`, `alarm_limits`: Low and high limits
* `display_precision`: The recommended number of decimal places to to display values with in user interfaces.

### Get multiple channels

If you pass a list of channel names to `.get()` you will get a list of data objects back:

```python
>>> x, y = archiver.get(['SR00TUM01:X_TUNE', 'SR00TUM01:Y_TUNE'], '2013-08-24 09:00', '2013-08-24 19:00')
>>> print x.values
[ 0.291, 0.290, ..., 0.289]
>>> print y.values
[ 0.216, 0.217, ..., 0.213]
```

### Times and timezones

The start and end times over which to fetch data can be `datetime`s or strings in ISO 8601 format (`yyyy-mm-dd HH-MM-SS`).

If no timezone is specified, UTC will be used. If a timezone is given, the returned channel data times will also be in this timezone.

```python
>>> import datetime
>>> import pytz
>>> tz = pytz.timezone('Australia/Melbourne')
>>> start = datetime.datetime(2012, 6, 1, tzinfo=tz)
>>> end = datetime.datetime(2012, 6, 30, tzinfo=tz)
>>> data = archiver.get('BR00EXS01:TUNNEL_TEMPERATURE_MONITOR', start, end)
                     time       value      status  severity
2012-08-09 11:00:46+10:00  23.8521546  HIHI_ALARM     MAJOR
2012-08-09 11:01:32+10:00  23.8737399  HIHI_ALARM     MAJOR
2012-08-09 11:02:19+10:00  23.8775618  HIHI_ALARM     MAJOR
...
```

### Interpolating

You can control how much data is returned from the archiver with the `limit` parameter. This is roughly equal to how many data points will be returned but the actual value will differ depending on how data is available and the interpolation method.

The interpolation is determined by the `interpolation` parameter. The allowed values are the constants available in `channelarchiver.codes.interpolation`: `RAW`, `SPREADSHEET`, `AVERAGED`, `PLOT_BINNING`, `LINEAR`. The default value is `LINEAR`.

```python
>>> from channelarchiver import codes
>>> channel = 'SR00MOS01:FREQUENCY_MONITOR'
>>> data = archiver.get(channel, '2012', '2013', limit=10000, interpolation=codes.interpolation.RAW)
```

