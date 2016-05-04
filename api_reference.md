#API Reference

This geolocation API is multi-threaded for maximum efficiency and reduced impact on user experience.

**IMPORTANT - READ THIS FIRST!** There are important differences between GPS Provider data, Network Provider data and Cellular data. It can be confusing. 

* GPS Provider data is focused on providing a device location using latitude and longitude information via a GPS.
* Network Provider data is focused on providing a device location using latitude and longitude via information from the cellular network.
* Cellular data is focused on providing cell tower meta-data via information from the cellular network. It may or may not contain a lat/lon. More info below.

##Methods
Method | Description
--- | ---
`start` | Starts any location providers that were specified in the configuration options. 
`stop` | Stops all location processes. This will also automatically occur when the app is placed in the background. The app will continue to consume memory.
`kill` | Shuts down all location activities, stops all threads and destroys the application instance. Can be used to hard stop a runaway GPS process, for example, or to simply close the application and stop all processes.

##Configuration Options (Required)

Option | Type | Description
--- | --- | ---
`minTime` | integer | The minimum time interval between location updates in milliseconds. Smaller numbers increase battery usage.
`minDistance` | integer | The minimum distance between location updates in meters. Smaller numbers increase battery usage.
`noWarn` | boolean | Display native warning popup dialog if GPS or Network is disabled.
`providers` | String | Acceptable values to specify location providers are: `"gps"`, `"network"`, `"cell"`, `"all"` or `"some"`. Network provider may return locations if WiFi or cellular internet is enabled.
`useCache` | boolean | Will return cached values from any active location provider. While not gauranteed, both GPS and NETWORK providers have a cache.
`satelliteData` | boolean | If `true` it returns all available satellite data from the GPS receiver. Requires that the `gps` provider is also enabled. <br><br>**CAUTION:** Activating satellite data will increase CPU and memory usage. 
`buffer` | boolean | If `true` it will start a buffer that returns the averaged geometric center of GPS and/or NETWORK locations. Use this when requirements call for determining a single, best location. The buffer uses a FIFO ordering, so new values added and old values are removed. 
`bufferSize` | integer | The maximum number of elements allowed within the buffer. It's strongly recommended to use as small of a buffer size as possible to minimize memory usage and garbage collection. Experiment to see what works best. This property will be ignored if `buffer` is set to `false`. Buffers larger than 30 elements may not be necessary.<br><br>**CAUTION:** Increasing the buffer size will increase CPU and memory usage. 

## GPS and Network Data

Whenever a location event is successful, this plugin will return the following location data in the form of a JSON payload. This section provides a description of the attribute/value pairs that are returned.

Example:

```javascript

	{
    "provider":"gps",
    "latitude":"42.05991886",
    "longitude":"-105.00000",
    "altitude":"1565.0",
    "accuracy":"9.0",
    "bearing":"0.0",
    "speed":"1.0",
    "timestamp":"1449876957000",
    "cached":"false"
    }

```

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `"gps"` or `"network"` | Let's you determine where this data is coming from.
`latitude` | String | number | Latitude in degrees. Be sure to check for `0.0`. values if no location was returned. `0.0` is a valid [WGS 84 location off the coast of Africa](https://en.wikipedia.org/wiki/Null_Island). 
`longitude` | String | number | Longitude in degrees. Be sure to check for `0.0`. values if no location was returned.
`altitude` | String | number | Altitude if available, in meters above the WGS 84 reference ellipsoid. If this location does not have an altitude then `0.0` is returned.
`accuracy` | String | number | Get the estimated accuracy of this location, in meters. <br><br>Android defines accuracy as the radius of 68% confidence. In other words, if you draw a circle centered at this location's latitude and longitude, and with a radius equal to the accuracy, then there is a 68% probability that the true location is inside the circle.<br><br>In statistical terms, it is assumed that location errors are random with a normal distribution, so the 68% confidence circle represents one standard deviation. Note that in practice, location errors do not always follow such a simple distribution.<br><br>This accuracy estimation is only concerned with horizontal accuracy, and does not indicate the accuracy of bearing, velocity or altitude if those are included in this Location.<br><br>If this location does not have an accuracy, then `0.0` is returned. All locations generated by the LocationManager include an accuracy.
`bearing` | String | number | The bearing, in degrees. Bearing is the horizontal direction of travel of this device, and is not related to the device orientation. It is guaranteed to be in the range (0.0, 360.0] if the device has a bearing.<br><br>If a location does not have a bearing then `0.0` is returned.
`speed` | String | number | The speed if it is available, in meters/second over ground. If this location does not have a speed then `0.0` is returned.
`timestamp` | String | number | Return the UTC time of this fix, in milliseconds since January 1, 1970.<br><br>Note that the UTC time on a device is not monotonic: it can jump forwards or backwards unpredictably.<br><br>All locations generated by the LocationManager are guaranteed to have a valid UTC time, however remember that the system time may have changed since the location was generated.
`cached` | String | boolean | Whether or not the location data, either GPS or Network, was returned from the device cache.<br><br>Note: cached data can be very unpredictable, use with caution and make sure to compare the timestamp with current time to establish the age of the cached location.

## Buffered GPS and Network Data

If you set the `buffer` configuration option to `true` this will enable new attribute/value pairs to be included in the return payload that are in addition to the GPS and Network elements listed above.

Buffering is typically used in commercial and government applications that require greater accuracy for determining static locations. Using the buffer will minimize location fluctuations by providing an averaged geometric center of cartesian coordinates.

Best practices for buffer requires the user to hold the device still in one location until the desired accuracy level is reached and the buffer is full. An all-JavaScript (non-native Android) sample app can be found in the html5-geolocation-tool-js [Field Location Template](https://github.com/Esri/html5-geolocation-tool-js).

Note that cached location data is not buffered since there will only one cached location per provider.

Example:

```javascript

    {
    "provider":"gps",
    "latitude":39.91974497,
    "longitude":-105.11730789,
    "altitude":1651,
    "accuracy":48,
    "bearing":0,
    "speed":0,
    "timestamp":1452634769000,
    "cached":false,
    "buffer":true,
    "bufferSize":10,
    "bufferedLatitude":39.919744632857146,
    "bufferedLongitude":-105.11730871142859,
    "bufferedAccuracy":48
    }

```

Property | Type |  Value | Description
--- | --- | --- | ---
`buffer` | String | boolean | Indicates whether or not buffering has been activated.
`bufferSize` | String | integer | Indicates the number of elements within the buffer. You can compare this value against the `maxBufferSize` as set in the configuration options.
`bufferedLatitude` | String | number | The buffer's geometric latitudinal center. Value is latitude in degrees. Be sure to check for `0.0` values indicating that no latitude data was provided.
`bufferedLongitude` | String | number | The buffer's geometric longitudinal center. Value is longitude in degrees. Be sure to check for `0.0` values indicating that no longitude data was provided.
`bufferedAccuracy` | String | number | The buffer's average horizontal accuracy in meters. It may be possible to have a buffered accuracy equal to `0.0`.

##Satellite Data

If you have the Configuration option `satelliteData` to `true`, then for each satellite detected by the GPS the following data will be returned as JSON in the callback. This section provides a description of the attribute/value pairs that are returned. 

Note: it's up to the GPS to determine whether or not any values will be provided for each property, this is especially true as the GPS warms up.

The plugin assigns a number to each satellites simply to make it easy to iterate through the JSON file. These numbers infer no actual relationship to a particular satellite. The `PRN` number is the actual satellite identifier.

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `"satellite"` | Let's you determine where this data is coming from.
`timestamp` | number | milliseconds | Time right now based on the Calendar whose locale is determined by system settings. Gregorian calendar assumes counting begins at the start of the epoch: i.e., YEAR = 1970, MONTH = JANUARY, DATE = 1, etc. For more info see [java.util.Calendar](http://developer.android.com/reference/java/util/Calendar.html).
number | String | number | Each satellite is assigned a sequential number for assisting with iterating the JSON. The numbers are simply a matter of convienence and have no other relationship with the satellite data.
`PRN` | String | integer | Returns the PRN (pseudo-random number) for the satellite. For more info see this [wikipedia article](https://en.wikipedia.org/wiki/List_of_GPS_satellites). Numbers 01 thru 32 represent GPS satellites. GLONASS uses higher numbers. 
`timeToFirstFix` | String | milliseconds | Returns the time required to receive the first fix since the most recent restart of the GPS engine.
`usedInFix` | String | boolean | Indicates whether or not a satellite was used to determine a GPS fix.
`azimuth` | String | number | Returns the azimuth of the satellite in degrees. The azimuth can vary between 0 and 360.
`elevation` | String | number | Returns the elevation of the satellite above the horizon in degrees. The elevation can vary between 0 and 90. 
`hasEphemeris` | String | boolean | Returns true if the GPS engine has ephemeris data for the satellite. 
`hasAlmanac` | String | boolean | Returns true if the GPS engine has almanac data for the satellite. 
`SNR` | String | number | Returns the signal to noise ratio for the satellite.  

# Cellular Data

If you have the `providers` Configuration option set to `cell` or `all` then this API will attempt to retrieve low-level data about the cellular service.

This API is non-specific in that it will attempt to return most cellular data that is available on the device. To clarify: 'most' cellular data means there may be some operational aspects of the native Android API that either aren't currently included or were unintentionally missed. 

Data will be returned under two circumstances:

* When first launched it will force a query via [TelephonyManager.getAllCellInfo()](http://developer.android.com/reference/android/telephony/TelephonyManager.html#getAllCellInfo()).
* When [PhoneStateListener.LISTEN_CELL_LOCATION](http://developer.android.com/reference/android/telephony/PhoneStateListener.html#LISTEN_CELL_LOCATION) indicates a change.

A full set of detailed information is available via the [`android.telephony`](http://developer.android.com/reference/android/telephony/package-summary.html) class documentation.

**IMPORTANT** Please make note of the following:

* There are minimum device SDK requirements. API level 17 is the current minimum to take advantage of this specific functionality. Plus, this project requires a minimum of SDK 21 or greater. Be aware of how you set the `minSdkVersion` in the AndroidManifest, for example: `<uses-sdk android:minSdkVersion="21" android:targetSdkVersion="22" />`
* Activating cellular data may result in additional network charges for the user.
* This information is not gauranteed.
* The `TelephonyManager` API may not work correctly on all devices. 
* To make use of this data you'll need access to a cell tower database. We don't provide cell tower location data, however there are databases and services available. One example provider is the [OpenCellId organization](http://wiki.opencellid.org/wiki/View_the_data). 
* Take extra steps to protect the input data when using this API. Check for `null` or `Integer.MAX_VALUE`.

Examples:

```javascript

	// cell_info wcdma - valid
	
	{
	"provider":"cell_info",
	"type":"wcdma",
	"timestamp":1461272187532,
	"cid":40052763,
	"lac":38995,
	"mcc":310,
	"mnc":410,
	"psc":217
	}
	
	// cell_location gsm - valid
	
	{
	"provider":"cell_location",
	"type":"gsm",
	"timestamp":1461274048131,
	"cid":40052763,
	"lac":38995,
	"psc":87
	}
	
	// cell_info wcdma - not valid
	// Problem: device returning max integer values.
	
	{
	"provider":"cell_info",
	"type":"wcdma",
	"timestamp":1461272187532,
	"cid":2147483647,
	"lac":2147483647,
	"mcc":2147483647,
	"mnc":2147483647,
	"psc":217
	}	

```


##cell_info CDMA Data

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `cell_info` | Let's you determine where this data is coming from.
`type` | String | `cdma` | Depends on the device, cell service provider and how many radios are active. It can return multiple values.
`timestamp` | number | milliseconds | Time right now based on the Calendar whose locale is determined by system settings. Gregorian calendar assumes counting begins at the start of the epoch: i.e., YEAR = 1970, MONTH = JANUARY, DATE = 1, etc. For more info see [java.util.Calendar](http://developer.android.com/reference/java/util/Calendar.html).
`baseStationId` | integer | 0 - 65535 | Base Station Id. Integer.MAX_VALUE if unknown.
`latitude` | number | +-90 | Decimal degrees. Integer.MAX_VALUE if unknown.
`longitude` | number | +-180 | Decimal degrees. Integer.MAX_VALUE if unknown.
`networkId` | integer | 0 - 65535 | Network Id. Integer.MAX_VALUE if unknown.
`systemId` | integer | 0 - 32767 | System Id. Integer.MAX_VALUE if unknown.

## cell_info LTE Data

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `cell_info` | Let's you determine where this data is coming from.
`type` | String | `lte` | Depends on the device, cell service provider and how many radios are active. It can return multiple values.
`timestamp` | number | milliseconds | Time right now based on the Calendar whose locale is determined by system settings. Gregorian calendar assumes counting begins at the start of the epoch: i.e., YEAR = 1970, MONTH = JANUARY, DATE = 1, etc. For more info see [java.util.Calendar](http://developer.android.com/reference/java/util/Calendar.html).
`ci` | integer | 0 - ~134,217,727 | 28-bit Cell Identity, Integer.MAX_VALUE if unknown.
`mcc` | integer | 0 - 999 | 3-digit Mobile Country Code. Integer.MAX_VALUE if unknown.
`mnc` | integer | 0 - 999 | 3-digit Mobile Network Code. Integer.MAX_VALUE if unknown.
`pci` | integer | 0 - 503 | Physical Cell Id. Integer.MAX_VALUE if unknown.
`tac` | integer | 0 - 65535 | 16-bit Tracking Area Code. Integer.MAX_VALUE if unknown.

## cell_info GSM Data

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `cell_info` | Let's you determine where this data is coming from.
`type` | String | `gsm` | Depends on the device, cell service provider and how many radios are active. It can return multiple values.
`timestamp` | number | milliseconds | Time right now based on the Calendar whose locale is determined by system settings. Gregorian calendar assumes counting begins at the start of the epoch: i.e., YEAR = 1970, MONTH = JANUARY, DATE = 1, etc. For more info see [java.util.Calendar](http://developer.android.com/reference/java/util/Calendar.html).
`cid` | integer | 0 - 268435455 | CID 28-bit UMTS Cell Identity described in TS 25.331. Integer.MAX_VALUE if unknown.
`lac` | integer | 0 - 65535 | 16-bit Location Area Code. Integer.MAX_VALUE if unknown.
`mcc` | integer | 0 - 999 | 3-digit Mobile Country Code. Integer.MAX_VALUE if unknown.
`mnc` | integer | 0 - 999 | 3-digit Mobile Network Code. Integer.MAX_VALUE if unknown.
`psc` | integer | 0 - 511 | 9-bit UMTS Primary Scrambling Code described in TS 25.331. Integer.MAX_VALUE if unknown.

## cell_info WCDMA Data

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `cell_info` | Let's you determine where this data is coming from.
`type` | String | `wcdma` | Depends on the device, cell service provider and how many radios are active. It can return multiple values.
`timestamp` | number | milliseconds | Time right now based on the Calendar whose locale is determined by system settings. Gregorian calendar assumes counting begins at the start of the epoch: i.e., YEAR = 1970, MONTH = JANUARY, DATE = 1, etc. For more info see [java.util.Calendar](http://developer.android.com/reference/java/util/Calendar.html).
`cid` | integer | 0 - 268435455 | CID 28-bit UMTS Cell Identity described in TS 25.331. Integer.MAX_VALUE if unknown.
`lac` | integer | 0 - 65535 | 16-bit Location Area Code. Integer.MAX_VALUE if unknown.
`mcc` | integer | 0 - 999 | 3-digit Mobile Country Code. Integer.MAX_VALUE if unknown.
`mnc` | integer | 0 - 999 | 3-digit Mobile Network Code. Integer.MAX_VALUE if unknown.
`psc` | integer | 0 - 511 | 9-bit UMTS Primary Scrambling Code described in TS 25.331. Integer.MAX_VALUE if unknown.

##cell_location CDMA Data

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `cell_location` | Let's you determine where this data is coming from.
`type` | String | `cdma` | Depends on the device, cell service provider and how many radios are active. It can return multiple values.
`timestamp` | number | milliseconds | Time right now based on the Calendar whose locale is determined by system settings. Gregorian calendar assumes counting begins at the start of the epoch: i.e., YEAR = 1970, MONTH = JANUARY, DATE = 1, etc. For more info see [java.util.Calendar](http://developer.android.com/reference/java/util/Calendar.html).
`baseStationId` | integer | ? | Base Station Id. -1 if unknown.
`latitude` | number | -90 - 90 | Decimal degrees. Integer.MAX_VALUE if unknown.
`longitude` | number | -180 - 180 | Decimal degrees. Integer.MAX_VALUE if unknown.
`networkId` | integer | ? | Network Id. -1 if unknown.
`systemId` | integer | ? | System Id. -1 if unknown.

## cell_location GSM Data

Property | Type |  Value | Description
--- | --- | --- | ---
`provider` | String | `cell_location` | Let's you determine where this data is coming from.
`type` | String | `gsm` | Depends on the device, cell service provider and how many radios are active. It can return multiple values.
`timestamp` | number | milliseconds | Time right now based on the Calendar whose locale is determined by system settings. Gregorian calendar assumes counting begins at the start of the epoch: i.e., YEAR = 1970, MONTH = JANUARY, DATE = 1, etc. For more info see [java.util.Calendar](http://developer.android.com/reference/java/util/Calendar.html).
`cid` | integer | ? | GSM cell id, -1 if unknown, 0xffff max legal value.
`lac` | integer | ? | GSM Location Area Code-1 if unknown, 0xffff max legal value.
`psc` | integer | ? | UMTS Primary Scrambling Code, -1 if unknown or GSM.



