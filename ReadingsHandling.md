> Note: Currently ignoring non-polled data RR cases.

> Note: Since no schema changes have been made the CR property Calibrate Previous Period is currently always false and RR closing total is always = RR total and the opening total is always equal to the "Next total" - RR period value.

# Calibration readings

Calibration readings are predominantly for use with pulse meters.

When a pulse meter displays a total value and outputs pulses to a pulse counter it is possible for the count displayed on the meter to differ from the count returned to DCS from the pulse counter. This can happen due to communications problems between the meter and the pulse counter, the pulse counter could have its value reset or simply because the meter and pulse counter are installed at different times.

To overcome any mismatch it is possible to enter calibration readings in DCS.
To enter a calibration reading the user must physically read the meter and record the value and the exact time.
When this calibration reading is entered into DCS it is used to calculate an offset between the value entered and the reading in the database at the corresponding time (linearly interpolated between half hour values). The offset is then applied to all readings when displayed or downloaded from the calibration reading's start time. It is possible to have any number of calibration readings for a given pulse meter. When more than one calibration reading exists all offsets are calculated and each offset is applied from the start time of the calibration reading until the start time of the next one.

At the start time of any calibration reading there is a boundary where readings before this time will have a different (or no) offset applied compared to after this time.
The question then arises how should the period value immediately before the boundary be displayed? Should the period value be displayed as the difference between the current and previous calibrated totals, or should this be ignored and the period value left unchanged (i.e. reflecting the difference between the current and previous uncalibrated readings. The user can control this by setting the To control this the Calibration Reading's Calibrate Previous Period value property.

## Example 1: a calibration reading applied from the beginning

The simplest example of a calibration reading is when an offset is applied to all readings for a meter. Consider the following calibration reading:

| Timestamp        | Reading value | Start time       | Calculated offset     |
| ---------------- | ------------- | ---------------- | --------------------- |
| 2000-01-01 01:15 | 1250          | 2000-01-01 00:00 | 1250 - 250 = 1000 [1] |

Notes

1. The offset for the calibration reading is calculated to be 1000. The linearly interpolated value of the uncalibrated reading at 2000-01-01 01:15 would be 250. Therefore the offset would be 1250 - 250. The offset is applied to all totals from the start time 2000-01-01 00:00.

| Reading time     | Uncalibrated total | Period value | Offset applied | Calibrated total | Period value |
| ---------------- | ------------------ | ------------ | -------------- | ---------------- | ------------ |
| 2000-01-01 00:00 | 0                  | 100          | 1000           | 1000             | 100          |
| 2000-01-01 00:30 | 100                | 100          | 1000           | 1100             | 100          |
| 2000-01-01 01:00 | 200                | 100          | 1000           | 1200             | 100          |
| 2000-01-01 01:30 | 300                | 100          | 1000           | 1300             | 100          |
| 2000-01-01 02:00 | 400                | 100          | 1000           | 1400             | 100          |
| 2000-01-01 02:30 | 500                | 100          | 1000           | 1500             | 100          |

## Example 2: a calibration reading applied to existing readings

A calibration reading's start time can be set to any reading (prior to the calibrations timestamp). Consider the following calibration reading where the start time is set to 01:00:

| Timestamp        | Reading value | Start time       | Calibrate Previous Period | Calculated offset |
| ---------------- | ------------- | ---------------- | ------------------------- | ----------------- |
| 2000-01-01 01:15 | 1250          | 2000-01-01 01:00 | not set/set               | 1250 - 350 = 900  |

| Reading time     | Uncalibrated total | Period value | Offset applied | Calibrated total | Period value    |
| ---------------- | ------------------ | ------------ | -------------- | ---------------- | --------------- |
| 2000-01-01 00:00 | 100                | 100          |                | 100              | 100             |
| 2000-01-01 00:30 | 200                | 100          |                | 200              | 100 or 1000 [1] |
| 2000-01-01 01:00 | 300                | 100          | 900            | 1200             | 100             |
| 2000-01-01 01:30 | 400                | 100          | 900            | 1300             | 100             |
| 2000-01-01 02:00 | 500                | 100          | 900            | 1400             | 100             |
| 2000-01-01 02:30 | 500                | 100          | 900            | 1500             | 100             |

Notes:

1. The period value can be presented as 100 if property Calibrate Previous Period is not set, or 1000 if it is set. 100 would represent the period value as it was if there was no calibration, while 1000 would be the period value as derived from the difference between the calibrated totals.

## Example 3: Multiple calibration readings

When multiple calibration readings exists the offsets are applied based on the start time. Consider the following:

| Timestamp        | Reading value | Start time       | Calibrate Previous Period | Calculated offset |
| ---------------- | ------------- | ---------------- | ------------------------- | ----------------- |
| 2000-01-01 01:15 | 1250          | 2000-01-01 00:00 | not set/set [1]           | 1250 - 250 = 1000 |
| 2000-01-01 03:15 | 1750          | 2000-01-01 03:00 | not set/set               | 1750 - 650 = 1100 |

Notes:

1. Since there are no readings prior to this point the value of this property doesn't matter.

| Reading time     | Uncalibrated total | Period value | Offset applied | Calibrated total | Calibrated period value |
| ---------------- | ------------------ | ------------ | -------------- | ---------------- | ----------------------- |
| 2000-01-01 00:00 | 0                  | 100          | 1000           | 1000             | 100                     |
| 2000-01-01 00:30 | 100                | 100          | 1000           | 1100             | 100                     |
| 2000-01-01 01:00 | 200                | 100          | 1000           | 1200             | 100                     |
| 2000-01-01 01:30 | 300                | 100          | 1000           | 1300             | 100                     |
| 2000-01-01 02:00 | 400                | 100          | 1000           | 1400             | 100                     |
| 2000-01-01 02:30 | 500                | 100          | 1000           | 1500             | 100 or 200 [1]          |
| 2000-01-01 03:00 | 600                | 100          | 1100           | 1700             | 100                     |
| 2000-01-01 03:30 | 700                | 100          | 1100           | 1800             | 100                     |
| 2000-01-01 04:00 | 800                | 100          | 1100           | 1900             | 100                     |
| 2000-01-01 04:30 | 900                | 100          | 1100           | 2000             | 100                     |

Notes:

1. The period value can be presented as 100 if property Calibrate Previous Period is not set, or 200 if it is set

## Example 4: multiple calibration readings added to existing readings with interpolation required

When interpolating readings immediately prior to a calibration reading start time the "Calibrate Previous Period" property is also taken into account. Consider the following calibration readings:

| Timestamp        | Reading value | Start time       | Calibrate Previous Period | Calculated offset |
| ---------------- | ------------- | ---------------- | ------------------------- | ----------------- |
| 2000-01-01 01:15 | 1250          | 2000-01-01 00:00 | not set/set               | 1250 - 250 = 1000 |
| 2000-01-01 03:15 | 1800          | 2000-01-01 03:00 | not set/set               | 1800 - 650 = 1150 |

| Reading time         | Uncalibrated total | Period value | Offset applied | Calibrated total | Calibrated period value |
| -------------------- | ------------------ | ------------ | -------------- | ---------------- | ----------------------- |
| 2000-01-01 00:00     | 0                  | 100          | 1000           | 1000             | 100                     |
| 2000-01-01 00:30     | 100                | 100          | 1000           | 1100             | 100                     |
| 2000-01-01 01:00     | 200                | 100          | 1000           | 1200             | 100                     |
| 2000-01-01 01:30     | 300                | _100_        | 1000           | 1300             | _100_ or _150_ [3]      |
| 2000-01-01 02:00 [1] | _400_              | _100_        | 1000           | _1450_ [2]       | _100_ or _150_ [3]      |
| 2000-01-01 02:30 [1] | _500_              | _100_        | 1000           | _1600_ [2]       | _100_ or _150_ [3]      |
| 2000-01-01 03:00     | 600                | 100          | 1150           | 1750             | 100                     |
| 2000-01-01 03:30     | 700                | 100          | 1150           | 1850             | 100                     |
| 2000-01-01 04:00     | 800                | 100          | 1150           | 1950             | 100                     |
| 2000-01-01 04:30     | 900                | 100          | 1150           | 2050             | 100                     |

Notes:

1. Missing readings at 2:00 and 2:30, values interpolated.
2. The calibrated readings are interpolated.
3. The period value can be presented as 100 if property Calibrate Previous Period is not set. Id set, the period values are calculated from the calibrated readings.

## Example 5: a pulse counter is changed or reset

Assume a meter physical pulse meter is working correctly as follows:

| Reading time     | Physical meter reading |
| ---------------- | ---------------------- |
| 2000-01-01 00:00 | 1100                   |
| 2000-01-01 00:30 | 1200                   |
| 2000-01-01 01:00 | 1300                   |
| 2000-01-01 01:30 | 1400                   |
| 2000-01-01 02:00 | 1500                   |
| 2000-01-01 02:30 | 1600                   |
| 2000-01-01 03:00 | 1700                   |
| 2000-01-01 03:30 | 1800                   |
| 2000-01-01 04:00 | 1900                   |
| 2000-01-01 04:30 | 2000                   |
| 2000-01-01 05:00 | 2100                   |

Between 02:30 and 03:00 the pulse counter is reset causing the count to go to 0. In DCS this would looks as follows:

| Reading time     | Total reading | Period value |
| ---------------- | ------------- | ------------ |
| 2000-01-01 00:00 | 1100          | 100          |
| 2000-01-01 00:30 | 1200          | 100          |
| 2000-01-01 01:00 | 1300          | 100          |
| 2000-01-01 01:30 | 1400          | 100          |
| 2000-01-01 02:00 | 1500          | -1500        |
| 2000-01-01 02:30 | 0             | 100          |
| 2000-01-01 03:00 | 100           | 100          |
| 2000-01-01 03:30 | 200           | 100          |
| 2000-01-01 04:00 | 300           | 100          |
| 2000-01-01 04:30 | 400           | 100          |
| 2000-01-01 05:00 | 500           | 100          |

At 04:15 a calibration reading was taken which would be applied from the start of the period when the counter was changed:

| Timestamp        | Reading value | Start time       | Calibrate Previous Period | Calculated offset |
| ---------------- | ------------- | ---------------- | ------------------------- | ----------------- |
| 2000-01-01 04:15 | 1950          | 2000-01-01 02:30 | SET                       | 1600              |

This would have a calculated off set off 1600 which would be applied from 02:30 onwards. In this case

| Reading time     | Uncalibrated total | Period value | Offset applied | Calibrated total | Period value |
| ---------------- | ------------------ | ------------ | -------------- | ---------------- | ------------ |
| 2000-01-01 00:00 | 1100               | 100          |                | 1100             | 100          |
| 2000-01-01 00:30 | 1200               | 100          |                | 1200             | 100          |
| 2000-01-01 01:00 | 1300               | 100          |                | 1300             | 100          |
| 2000-01-01 01:30 | 1400               | 100          |                | 1400             | 100          |
| 2000-01-01 02:00 | 1500               | -1500 [1]    |                | 1500             | 100 [2]      |
| 2000-01-01 02:30 | 0                  | 100          | 1600           | 1600             | 100          |
| 2000-01-01 03:00 | 100                | 100          | 1600           | 1700             | 100          |
| 2000-01-01 03:30 | 200                | 100          | 1600           | 1800             | 100          |
| 2000-01-01 04:00 | 300                | 100          | 1600           | 1900             | 100          |
| 2000-01-01 04:30 | 400                | 100          | 1600           | 2000             | 100          |
| 2000-01-01 05:00 | 500                | 100          | 1600           | 2100             | 100          |

1. If no calibration is performed the period value would be 0-1500.
2. Since the Calibrate Previous Period property is set the period value is 1600-1500. If the property is not set this value would be -1500 which would not reflect the consumption for this period.

# Register resets

Register resets are predominantly for use with intelligent meters and Modbus meters. They are not normally used for pulse meters.

When a meter is changed, or there is a rollover in the actual meter, there may be a discontinuity in the readings in DCS.

Register resets are defined in terms of a closing total and an opening total and the reset is considered to take place at the start of a period.
The main purpose of a Register Reset is to explicitly mark a point in time when a physical meter was changed and to use the opening and closing totals to calculate the period values and when interpolation is required.

## Example 1:a physical meter is (nominally) reset at 1:30 and its value is set to zero

This may look like this in DCS

| Time  | Total | Period value |
| ----- | ----- | ------------ |
| 00:00 | 100   | 100          |
| 00:30 | 200   | 100          |
| 01:00 | 300   | -300 [1]     |
| 01:30 | 0     | 100          |
| 02:00 | 100   | 100          |
| 02:30 | 200   | 100          |
| 03:00 | 300   | 100          |
| 03:30 | 400   | 100          |

Notes:

1. The period value is shown as -300 since this is the difference between the total at 01:30 and 01:00 whereas in reality 100 units were consumed in that period

To handle this case a Register reset could be introduced with the following values:

| Timestamp | Closing total | Opening total |
| --------- | ------------- | ------------- |
| 01:30     | 400           | 0             |

This would mean that the readings in DCS would look like this

| Time  | Total | Period value |
| ----- | ----- | ------------ |
| 00:00 | 100   | 100          |
| 00:30 | 200   | 100          |
| 01:00 | 300   | 100 [1]      |
| 01:30 | 0 [2] | 100 [3]      |
| 02:00 | 100   | 100          |
| 02:30 | 200   | 100          |
| 03:00 | 300   | 100          |
| 03:30 | 400   | 100          |

Notes:

1. This is the difference between the closing total from the RR and the total at 01:00.
2. This is the opening total from the RR
3. This is calculated as the difference the total a 02:00 and the opening reading from the RR

## Example 2: a physical meter is powered down just after 01:00 and a replacement (with registers set to 0) is started just before 3:00. Missing data is interpolated using a Register Reset

This may look like this in DCS

| Time  | Total | Period value | Interpolated total value | Interpolated period value |
| ----- | ----- | ------------ | ------------------------ | ------------------------- |
| 00:00 | 1100  | 100          | 1100                     | 100                       |
| 00:30 | 1200  | 100          | 1200                     | 100                       |
| 01:00 |       |              | _1300_                   | _-320_ [1]                |
| 01:30 |       |              | _980_                    | _-320_                    |
| 02:00 |       |              | _660_                    | _-320_                    |
| 02:30 |       |              | _340_                    | _-320_                    |
| 03:00 | 20    | 100          | 20                       | 100                       |
| 03:30 | 120   | 100          | 120                      | 100                       |

Notes:

1. The period value is shown as -320 since this is the difference between the total at 01:00 and 03:00 divided by 4, whereas in reality approximately 100 units were consumed in per period

To handle this case a Register reset could be introduced with the following values:

| Timestamp | Closing total | Opening total |
| --------- | ------------- | ------------- |
| 03:00     | 1700          | 20            |

This would mean that the readings in DCS would look like this

| Time  | Total | Period value | Interpolated total value | Interpolated period value |
| ----- | ----- | ------------ | ------------------------ | ------------------------- |
| 00:00 | 1100  | 100          | 1100                     | 100                       |
| 00:30 | 1200  | 100          | 1200                     | 100                       |
| 01:00 |       |              | _1300_                   | _100_ [1]                 |
| 01:30 |       |              | _1400_                   | _100_                     |
| 02:00 |       |              | _1500_                   | _100_                     |
| 02:30 |       |              | _1600_                   | _100_                     |
| 03:00 | 20    | 100          | 20                       | 100                       |
| 03:30 | 120   | 100          | 120                      | 100                       |

Notes:

1. 1. The period value is shown as 100 since this is the difference between the total at 01:00 and closing total of the Register Reset divided by 4

## Calibration readings and Register resets

It is not normally required to have calibration readings and register resets applied to the same readings but it is not prohibited.

Note that if one or more Register Resets exists for the register that Calibration Readings are applied **after** Register Resets (so Register Reset values must be in non-calibrated values). Also, the offset of a calibration reading will **not** continue to be used past a register reset as the offset would no longer be valid in such a case.
