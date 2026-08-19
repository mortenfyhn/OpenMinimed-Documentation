# Continuous Glucose Monitoring Service

This service provides access to data from a CGM sensor (such as the Guardian 4) that is connected to the pump. Its core components is the _CGM Measurement_ characteristic which contains the actual sensor measurements. Configure notifications for this characteristic. The pump will then automatically send new CGM measurements as they come in from the sensor. Additionally, specific data records can be requested through the service's _Record Access Control Point_ characteristic and will also be delivered as notifications of the _CGM Measurement_ characteristic.


## CGM Feature

This characteristic can be read to retrieve pump features regarding the CGM. Most notably the _E2E-CRC Supported_ bit. If it is set, the optional _E2E-CRC_ field in the other characteristics of this service must be included and populated with a CRC over the data. See [[CGMS, sec. 3.11]](#ref-cgms) for the specifics. Note that this goes both ways: The CRC must be included in all our requests, and the pump will also add it in its responses.

The _E2E-CRC Supported_ bit seems to be always set for a 780G pump, which is why we explicitly mentiond it here.

The implementation follows the standard defined in [[CGMS]](#ref-cgms) and [[GSS, sec. 3.42]](#ref-gss) without any changes additions, so we will not reproduce that information here.

Note that the data returned by the pump in this characteristic is _not_ SAKE-encrypted.

The spec requires the _CGM Feature_ to be static during a connection. So reading it _once_ is sufficient. The features will not change in the middle of a connection.


## CGM Measurement

Values from a connected CGM sensor are reported in this characteristic. Also included are optional information such as trend information or warnings regarding the glucose level.

The implementation follows the standard defined in [[CGMS]](#ref-cgms) and [[GSS, sec. 3.43]](#ref-gss) without any changes or additions, so we will not reproduce that information here.

The data returned by the pump in this characteristic is SAKE-encrypted.

On a 780G, while the sensor has no displayable glucose the pump keeps sending measurement records with a plain glucose value of **0 mg/dL** and advancing time offsets, rather than the SFLOAT NaN/NRes/Inf sentinels. This holds for SG below the display range (_Sensor Message State_ `0x09`, pump shows "LO"), "sensor updating" (`0x02`), "change sensor" (`0x07`) and warm-up (`0x08`) — a full sensor-change arc produced no sentinel at any point. A new session's time offsets restart near zero and the 0-records advance in the usual 5-minute steps; while the transmitter is off charging, no records flow at all. A client that treats 0 as a real reading will graph false zeros — check the _Sensor Message State_ in [IDD Status](idd-service.md) to tell off-scale from no-data. SG above the display range (`0x0A`) is expected to behave the same but has not been captured.


## Session Start Time

This characteristic provides the absolute time of the first CGM measurement taken. This serves as reference for the measurements reported in the _CGM Measurement_ characteristic which only contain a relative offset to this start time.

The implementation follows the standard defined in [[CGMS]](#ref-cgms) and [[GSS, sec. 3.45]](#ref-gss) without any changes or additions, so we will not reproduce that information here.

The data returned by the pump in this characteristic is SAKE-encrypted.


## Session Run Time

This characteristic provides the expected run time of the CGM session.

The implementation follows the standard defined in [[CGMS]](#ref-cgms) and [[GSS, sec. 3.44]](#ref-gss) without any changes or additions, so we will not reproduce that information here.

The data returned by the pump in this characteristic is **NOT** SAKE-encrypted.


## CGM Status

This characteristic reports the current status of the CGM sensor. It contains the same status bits as the _Sensor Status Annunciation_ field in the _CGM Measurement_ characteristic but does not require a running CGM session that is reporting measurements.

The implementation follows the standard defined in [[CGMS]](#ref-cgms) and [[GSS, sec. 3.47]](#ref-gss) without any changes or additions, so we will not reproduce that information here.

The data returned by the pump in this characteristic is SAKE-encrypted.


## CGM Specific Ops Control Point (SOCP)

A command (identified by its _opcode_) is sent by writing to this characteristic. The pump responds by sending an indication for the same characteristic.

The written data must be SAKE-encrypted. The returned data is also SAKE-encrypted.

The pump supports the following standard command as defined in [[CGMS]](#ref-cgms) and [[GSS, sec. 3.46]](#ref-gss):

* _Get Glucose Calibration Value_ (opcode 0x05)

Additionally, the following custom commands are supported:


### Format of custom _Get Calibration Context_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x81   | 1             | None
Record Number                | u16          | 2             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x82   | 1             | None
Status                       | u8           | 1             | None
Calibration Factor           | u16          | 2             | ???
E2E-CRC                      | u16          | 0 or 2        | None

Bits in the _Status_ field are defined as follows:

Bit | Definition                         | Description
----|------------------------------------|-------------
0   | Change Sensor Needed               |
1   | No Further Calibrations            |


### Format of custom _Read Session Start Time_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x83   | 1             | None
Session ID                   | u16          | 2             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x84   | 1             | None
Session ID                   | u16          | 2             | None
Date Time                    | see note-1   | 7             | see note-1
Time Zone                    | see note-2   | 1             | see note-2
DST Offset                   | see note-3   | 1             | see note-3
E2E-CRC                      | u16          | 0 or 2        | None

note-1: See [[GSS, sec. 3.79]](#ref-gss) for the definition of this type.

note-2: See [[GSS, sec. 3.255]](#ref-gss) for the definition of this type.

note-3: See [[GSS, sec. 3.86]](#ref-gss) for the definition of this type.


### Format of custom _Read Current Session ID_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x8c   | 1             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x8d   | 1             | None
Session ID                   | u16          | 2             | None
E2E-CRC                      | u16          | 0 or 2        | None


### Format of custom _Get Sensor Details_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x90   | 1             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x91   | 1             | None
Flags                        | 8 bit        | 1             | None
Annunciation                 | 16 bit       | 0 or 2        | None
Maximum Calibration Interval | u16          | 0 or 2        | ???
Maximum Sensor Life          | u16          | 0 or 2        | minutes
Sensor Flex Package Version  | u16          | 0 or 2        | ???
Warm-Up Period               | u8           | 0 or 1        | ???
E2E-CRC                      | u16          | 0 or 2        | None

Bits in the _Flags_ field are defined as follows:

Bit | Definition                   | Description
----|------------------------------|-------------
0   | Sensor Details Annunciation  | If this bit is set, field _Annunciation_ is present
1   | Maximum Calibration Interval | If this bit is set, field _Maximum Calibration Interval_ is present
2   | Maximum Sensor Life          | If this bit is set, field _Maximum Sensor Life_ is present
3   | Sensor Flex Package Version  | If this bit is set, field _Sensor Flex Package Version_ is present
4   | Sensor Warm-Up Period        | If this bit is set, field _Warm-Up Period_ is present

Bits in the _Annunciation_ field are defined as follows:

Bit | Definition                         | Description
----|------------------------------------|-------------
0   | Approved Treatment                 |
1   | Disposable                         |
2   | Calibration-Free                   |
3   | Has Calibration Recommended        |
4   | Has Abnormal SG Increase Detection |
5   | Calibration Transfer Supported     |


## Time Of Sensor Expiration

Field Name                   | Data Type    | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Value                        | u16          | 2             | minutes


## Time Of Next Calibration Recommended

Field Name                   | Data Type    | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Timestamp                    | u32          | 4             | ???
Clock ID                     | u32          | 4             | None
Time Offset                  | u16          | 2             | ???


## References

<a id="ref-cgms"></a>
**[CGMS]**
[Continuous Glucose Monitoring Service](specs/CGMS_v1.0.2.pdf), Bluetooth® Service Specification, v1.0.2

<a id="ref-gss"></a>
**[GSS]**
[GATT Specification Supplement (GSS)](specs/GATT_Specification_Supplement.pdf), Bluetooth® Document, 2025-12-23
