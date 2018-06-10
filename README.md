# Dynamic Pricing Communication

# Protocol format
The protocol’s format is defined for JSON format.

The parking data included in the protocol’s format, are referred to as elements. The elements are further divided into three categories: location, occupancy, and tariff. The three categories act as root objects and hold the other elements together. The root elements are separated into three different arrays under a parent object. Arrays are used for the root element as there might be multiple occurrences of each type. Any object that has multiple occurrences, such as the root objects, include a unique identification string. The three categories and what they include is explained in the following subsections.

## Tariff
Tariff data is the object that describes the parking fees of a parking location. Tariff data is included in the protocol to enable fast and easy sharing of new tariffs. There exists a wide variety of how a tariff can be expressed. The proposed protocol aims to express both the simplest and the most complex tariff while maintaining a generic method of expressing tariffs. 

The tariff information is divided into three parts: restriction, rate, and schedule data. Because each location can have multiple tariff objects, each tariff must be provided with a globally unique tariff ID and mapped with a location ID.  

### Tariff Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|tariffId|String(ID)|Unique ID of the tariff||
|locationId|String(ID)|ID of the location the tariff is meant for||
|restriciton|Restriction|Object containint restrictions for the tariff||
|rate|Array(Rate)|Rate components||
|activeSchedule|Array(ActiveSchedule)|Schedules defining the paid hours||
|validSchedule|Array(ValidSchedule)|Schedules defining when the tariff is valid||
|log|Log|Information regarding when the tariff object was created and latest updated||

### Restriction
Restrictions object includes information that limits or provide additional information regarding a tariff. 

The elements that restricts tariffs are: _argetGroup_, _vehicles_, _maxParkingTime_, _maxPaidParkingTime_, _maxFee_, and _minFee_.  All these elements are optional to include in the format. Therefore, if any of the elements are unassigned it means that the tariff is not restricted in any of the specific ways. 

The elements that provides additional information for a tariff are: _tariffType_, _prepaid_, and _resetTime_. The optional elements are prepaid, and resetTime. The prepaid element should be false by default and if no resetTime is defined, it means that the rates will never reset, unless the driver restarts the parking themselves.

#### Restriction Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|tariffType|TariffType|What kind of tariff||
|maxFee|Float|Overall maximum fee for this tariff <br/> Might be restricted by max rates instead||
|minFee|Float|Minimum fee a driver need to pay||
|maxPaidParkingTime|Int|Maximum parking time counting paid hours||
|maxParkingTime|Int|Overall maximum parking time||
|prepaid|Boolean|The fees are paid in advance||
|resetTime|Int|The time next day begins in minutes|Min value: 0 <br/> Max value: 1440|
|targetGroup|Array(ParkingType)|Tokens if the tariff is exclusive to some groups||
|vehicles|Array(VehicleType)|Tokens if the tariff is exclusive to some vehicle types||

#### TariffType Tokens
|Token Name|Description|
|----------|-----------|
|REGULAR|The tariff has stoppable rates|
|FIXED|The tariff only has fixed rates|
|EVENT|???|
|CONTRACT||

#### ParkingType Tokens
|Token Name|Description|
|----------|-----------|
|PRIVATE|Private tariff|
|PERSONNEL|Tariff is for personnel only|
|RESIDENTIAL|Tariff is for the residents in the area|
|INDIVIDUAL|The tariff targetse a single|
|PUBLIC|The tariff is open to the public|

#### VehicleType Tokens
|Token Name|Description|
|----------|-----------|
|CAR||
|VAN||
|MC||
|TRUCK||
|ELECTRIC||
|BUS||

### Rate
The rate is the smallest component of a parking fee, which describes the price development. The rate component describes the fee as a value over a time interval in minutes. For more advanced price strategies, a tariff can consist of multiple rate components. Therefore, each rate must a number defining in which order the rates applies, which also can be used to differentiate the rates.

#### Rate Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|order|Int|Unique number defining the priority of the rate. <br />Lower equals higher priority|Min value: 0|
|value|Float|Number defining the fee|Min value: 0|
|interval|Int|Number defining the period|Min value: 1|
|intervals|Int|Number defining how many times the period can be repeated|Min value: 1|
|unit|UnitType|Token defining the time unit of interval|Default: MIN|
|repeat|Boolean|Set true if the rate should be repeated infinitely|Default: false|
|max|Boolean|Set true if the rate defines a max fee|Default: false|
|countOnlyPaidTime|Boolean|Set true if the interval only should only count for paid hours|Default: false|

#### UnitType Tokens
|Token Name|Description|
|----------|-----------|
|MIN|The rate interval is given in minutes|
|DAY|The rate interval is given in days|
|WEEK|The rate interval is given in weeks|
|MONTH|The rate interval is given in monts|
|YEAR|The rate interval is given in years|

The simplest tariff with a constant cost per time units can be expressed using one rate component. However, a tariff can be more advanced than a constant cost per time unit, which requires more than 1 rate component to express the tariff.
For example: see **Rate example 1**, which describes a pricing strategy: the first-hour costs 10 SEK, the second hour is free, and thereafter the cost is 20 SEK per hour. Because no unit was provided in the rates, the interval is assumed to be in minutes. 

##### Rate example 1
```
{
 "rates":[
  {"order":0, "value":10, "interval":60, "intervals":1, "repeat":false},
  {"order":1, "value":0,  "interval":60, "intervals":1, "repeat":false},
  {"order":2, "value":20, "interval":60, "intervals":1, "repeat":true}
]}
```

Apart from consisting of several rate components, a complex tariff might also include both regular and fixed rates. In such case, the fixed rates must be expressed differently than the regular rates. For example: see **Rate example 2**, which describes the pricing strategy: the first hour (1 + 59 minutes) has a fixed rate of 15 SEK, and thereafter the cost is 10 SEK per hour. 

##### Rate example 2
```
{
 "rates":[
  {"order":0, "value":15, "interval":1,  "intervals":1, "repeat":false},
  {"order":1, "value":0,  "interval":59, "intervals":1, "repeat":false},
  {"order":2, "value":10, "interval":60, "intervals":1, "repeat":true}
]}
```

Apart from complex tariffs, a tariff might also have maximum fees such as daily, weekly, or monthly. However, a day, week, or month can be defined differently. Therefore, the rate object includes the element unit which allows to define the date time unit. In the table below possible date time definintions are shown. Using the rate objects, it is, therefore, possible to define different types of max fees. Max fees are recognized by the element max being set to true. For example: see **Rate example 3**, which describes the pricing strategy: 10 SEK per hour, with a daily maximum fee of 60 SEK (full day, i.e. 24h), and a weekly  maximum fee of 200 (full week, i.e. 168h). 

##### Possible date time definitions
|Value unit|  MIN  | Calendar Days | Calendar Weeks | Calendar Months | Calendar Years |
|----------|----:|----:|----:|----:|----:|
|1 Hour | 60|-|-|-|-|
|1 Day | 1 440|1|-|-|-|
|1 Week | 10 080|7|1|-|-|
|1 Month | 40 320 <br /> to 44 640|28 <br /> to 31| ~4|1|-|
|1 Year | 525 600 <br /> to 527 040| 365 <br /> to 366|~52|12|1|

##### Rate example 3
```
{
 "rates":[
  {"order":1,"value":10,"interval":60,"intervals":1,"repeat":true},
  {"order":2,"value":60,"interval":1440,"intervals":1,"max":true},
  {"order":3,"value":200,"interval":10080,"intervals":1,"max":true}
]}
```

### Schedules
There are two types of schedule objects: active- and validSchedule. The activeSchedule is used to define when a tariff is active, and restricts a tariff based on the elapsed- and current time. An activeSchedule can be interpreted as rules to when a tariff is applicable. This is necessary to be able to define time periods when the parking is free. The activeSchedule are also used to differentiate which tariff to use when a single location has multiple tariffs. For example, parking after a specific time might have a cheaper tariff or be free. The validSchedule, on the other hand, restrict when a tariff is valid and usable based on when the parking started. The validSchedule is necessary when defining early bird tariffs, where a driver get cheaper rates if the driver starts parking inside a specific time period.

The time values (_startTime_, _endTime_, _validTimeFrom_, and _validTimeTo_) have a time accuracy in minutes, and are given in minutes past midnight. To prevent time values from clashing, some standard definitions exists. The _startTime_ and _validTimeFrom_ are defined as the exact second the minutes defines, for example, 60 (01:00) defines the start time 01:00:00. In contrast, _endTime_ and _validTimeTo_ are defined as 1 second before what the minutes defines, for example, 1380 (23:00) defines the end time 22:59:59.

#### ActiveSchedule Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|activeScheduleId|String(ID)|Unique ID||
|startTime|Int|Start time of the paid hours |Min value: 0 <br />Max value: 1440|
|endTime|Int|End time of the paid hours|Min value: 0 <br />Max value: 1440|
|days|Array(Days)|The days the tariff are active, i.e. days with parking fees |Only unique days|

An activeSchedule might need more than one schedule unit to express the intention because the active time might vary per day. Therefore, each schedule requires a locally unique ID. For example, Active Schedule example shows a tariff is active during from 07:00 to 17:00 (16:59:59) weekdays and from 07:00 to 14:00 (13:59:59) Saturdays, and thus it requires at least two schedule units to describe. Note that there is no time defined for Sundays, hence, there is no parking fee during Sundays.

##### Active Schedule example
```
{"activeSchedules":[
  {"activeScheduleId":"as-0",
   "startTime":420, 
   "endTime":1020,
   "days": ["MONDAY","TUESDAY","WEDNESDAY","THURSDAY","FRIDAY"] },
  {"activeScheduleId":"as-1",
   "startTime":420, 
   "endTime":840,
   "days": ["SATURDAY"] },
]}
```

#### ValidSchedule Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|validScheduleId|String(ID)|Unique ID||
|validFrom|DateTime|Date defining when the tariff starts being valid<br />i.e. can be used||
|validTo|DateTime|Date defining when the tariff stop being valid<br />i.e. cannot be used anymore||
|validTimeFrom|Int|The time when the tariff can be started|Min value: 0 <br />Max value: 1440|
|validTimeTo|Int|The time when the tariff cannot be started anymore|Min value: 0 <br />Max value: 1440|
|validDays|Arrays(Day)|The days the tariff can be started|Only unique days|

Like activeSchedules, multiple validSchedules might be necessary, and therefore locally unique IDs is required. For example: see Figure 16, which says that a tariff can be started between 1 January 2018 to 30 June 2018, Mondays to Fridays, from 00:00 to 09:00 (08:59:59), and from 21:00 to 00:00 (23:59:59). 

##### Valid Schedule example
```
{"validSchedules":[
  {"validScheduleId":"vs-0",
   "validFrom":"2018-01-01T00:00:00", 
   "validTo":"2018-06-30T23:59:59",
   "validTimeFrom":0, 
   "validTimeTo":540,
   "validDays": ["MONDAY","TUESDAY","WEDNESDAY","THURSDAY","FRIDAY"]
  },
  {"validScheduleId":"vs-1",
   "validFrom":"2018-01-01T00:00:00", 
   "validTo":"2018-06-30T23:59:59",
   "validTimeFrom":1260, 
   "validTimeTo":1440,
   "validDays": ["MONDAY","TUESDAY","WEDNESDAY","THURSDAY","FRIDAY"]
  }
]}
```

#### Day Tokens
|Token Name|Description|
|----------|-----------|
|MONDAY||
|TUESDAY||
|WEDNESDAY||
|THURSDAY||
|FRIDAY||
|SATURDAY||
|SUNDAY||
|HOLIDAY|Also known as red days|
|DAY_BEFORE_RED_DAY|Also know as day before holiday|


## Location

## Occupancy