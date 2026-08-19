# Pump GATT services

The 700-series pumps expose the following services:


## Generic Access Profile (GAP)

UUID   | Type            | Notes
-------|-----------------|-------
0x1800 | Primary Service |

This is the standard GATT service as defined by the Bluetooth SIG, containing only the mandatory characteristics.

### Service characteristics

UUID   | Name                       | Properties | SAKE-encrypted | Notes
-------|----------------------------|------------|----------------|-------
0x2a00 | Device Name                | Read       | no             |
0x2a01 | Appearance                 | Read       | no             |


## Device Information

UUID   | Type            | Notes
-------|-----------------|-------
0x180a | Primary Service |

This is the standard GATT service as defined by the Bluetooth SIG.

### Service characteristics

UUID   | Name                                                | Properties | SAKE-encrypted | Notes
-------|-----------------------------------------------------|------------|----------------|-------
0x2a29 | Manufacturer Name String                            | Read       | no             |
0x2a24 | Model Number String                                 | Read       | no             |
0x2a25 | Serial Number String                                | Read       | no             |
0x2a27 | Hardware Revision String                            | Read       | no             |
0x2a26 | Firmware Revision String                            | Read       | no             |
0x2a28 | Software Revision String                            | Read       | no             |
0x2a23 | System ID                                           | Read       | no             |
0x2a50 | PnP ID                                              | Read       | no             |
0x2a2a | IEEE 11073-20601 Regulatory Certification Data List | Read       | no             |


## Insulin Delivery

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00000100-0000-1000-0000-009132591325 | Primary Service |

This service is based on the standard GATT service _Insulin Delivery Service_ (IDS, UUID 0x183A) defined by the Bluetooth SIG. It is actually comprised of the same characteristics but with Medtronic's own UUIDs. The only exceptions being the _IDD Record Access Control Point_ (UUID 0x2b27) which is replaced by the _Record Access Control Point_ (UUID 0x2a52) and the addition of a custom _GST Battery Level_ characteristic.

It provides access to the pump's status and historical event data (sensor values, boluses etc.).

### Service characteristics

UUID                                 | Name                                  | Properties      | SAKE-encrypted | Notes
-------------------------------------|---------------------------------------|-----------------|----------------|-------
00000101-0000-1000-0000-009132591325 | IDD Status Changed                    | Read, Indicate  | yes            |
00000102-0000-1000-0000-009132591325 | IDD Status                            | Read            | yes            |
00000103-0000-1000-0000-009132591325 | IDD Annunciation Status               | Read            | yes            |
00000104-0000-1000-0000-009132591325 | IDD Features                          | Read            | yes            |
00000105-0000-1000-0000-009132591325 | IDD Status Reader Control Point       | Write, Indicate | yes            |
00000106-0000-1000-0000-009132591325 | IDD Command Control Point             | Write, Indicate | yes            |
00000107-0000-1000-0000-009132591325 | IDD Command Data                      | Notify          | yes            |
0x2a52                               | Record Access Control Point           | Write, Indicate | no             |
00000108-0000-1000-0000-009132591325 | IDD History Data                      | Notify          | yes            |
00000400-0000-1000-0000-009132591325 | GST Battery Level                     | Read, Notify    | no             |


## Current Time

UUID   | Type            | Notes
-------|-----------------|-------
0x1805 | Primary Service |

This is the standard GATT service as defined by the Bluetooth SIG, containing only the mandatory characteristic.

### Service characteristics

UUID   | Name                       | Properties   | SAKE-encrypted | Notes
-------|----------------------------|--------------|----------------|-------
0x2a2b | Current Time               | Read, Notify | no             |


## Battery

UUID   | Type            | Notes
-------|-----------------|-------
0x180f | Primary Service |

This is the standard GATT service as defined by the Bluetooth SIG, containing only the mandatory characteristic.

### Service characteristics

UUID   | Name                       | Properties   | SAKE-encrypted | Notes
-------|----------------------------|--------------|----------------|-------
0x2a19 | Battery Level              | Read, Notify | no             | Coarse on the 780G — see note.

> On a 780G, `0x2A19` is coarse: only 50% (flat across a day) and 100% (fresh battery) have been
> observed, with no samples taken while the battery drained. For a reliable low-battery signal, use
> the pump's "replace battery" annunciation (type `0x054`) instead.


## Continuous Glucose Monitoring

UUID   | Type            | Notes
-------|-----------------|-------
0x181f | Primary Service |

This is an implementation of the homonymous service defined by the Bluetooth SIG. It adds a couple of characteristics and SAKE encryption for most of their data.

### Service characteristics

UUID                                 | Name                                  | Properties      | SAKE-encrypted | Notes
-------------------------------------|---------------------------------------|-----------------|----------------|-------
0x2aa7                               | CGM Measurement                       | Notify          | yes            |
0x2aa8                               | CGM Feature                           | Read            | no             |
0x2aa9                               | CGM Status                            | Read            | yes            |
0x2aaa                               | CGM Session Start Time                | Read, Write     | yes            |
0x2aab                               | CGM Session Run Time                  | Read            | no             |
0x2a52                               | Record Access Control Point           | Write, Indicate | no             |
0x2aac                               | CGM Specific Ops Control Point        | Write, Indicate | yes            |
00000200-0000-1000-0000-009132591325 | CGM Measurement (Medtronic Extension) | Read            | ?              |
00000201-0000-1000-0000-009132591325 | Sensor Connected State                | Read, Indicate  | ?              |
00000202-0000-1000-0000-009132591325 | Time Of Sensor Expiration             | Indicate        | no             |
00000203-0000-1000-0000-009132591325 | Sensor Calibration Time               | Indicate        | ?              |
00000204-0000-1000-0000-009132591325 | Time Of Next Calibration Recommended  | Read, Indicate  | no             |
00000205-0000-1000-0000-009132591325 | Algorithm Data                        | Read            | ?              |


## History and Trace

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00000300-0000-1000-0000-009132591325 | Primary Service |

This is a Medtronic custom GATT service.

### Service characteristics

UUID                                 | Name                                             | Properties      | SAKE-encrypted | Notes
-------------------------------------|--------------------------------------------------|-----------------|----------------|-------
00000360-0000-1000-0000-009132591325 | Repository And Transfer Management Control Point | Write, Indicate | no             |
00000350-0000-1000-0000-009132591325 | Slice Record                                     | Notify          | no             |
0x2a52                               | Record Access Control Point                      | Write, Indicate | no             |
00000370-0000-1000-0000-009132591325 | Repository Management Control Point SE           | Write, Indicate | no             |


## Network Operational State

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00000500-0000-1000-0000-009132591325 | Primary Service |

This is a Medtronic custom GATT service.

### Service characteristics

UUID                                 | Name     | Properties          | SAKE-encrypted | Notes
-------------------------------------|----------|---------------------|----------------|-------
00000510-0000-1000-0000-009132591325 | NOS Port | Read, Write, Notify | ?              |


## ???

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00001020-0000-1000-0000-009132591325 | Primary Service |

This is a Medtronic custom GATT service.

### Service characteristics

UUID                                 | Name                          | Properties      | SAKE-encrypted | Notes
-------------------------------------|-------------------------------|-----------------|----------------|-------
00001010-0000-1000-0000-009132591325 |                               | Write, Indicate | ?              |


## Secure Session Establishment

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00000700-0000-1000-0000-009132591325 | Primary Service |

A Medtronic custom service.

### Service characteristics

UUID                                 | Name              | Properties                 | SAKE-encrypted | Notes
-------------------------------------|-------------------|----------------------------|----------------|-------
00000701-0000-1000-0000-009132591325 | SSE Control Point | Write, Indicate            | ?              |
00000702-0000-1000-0000-009132591325 | SSE Data          | Write w/o response, Notify | ?              |


## Certificate Management

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00000600-0000-1000-0000-009132591325 | Primary Service |

A Medtronic custom service.

### Service characteristics

UUID                                 | Name                                 | Properties                 | SAKE-encrypted | Notes
-------------------------------------|--------------------------------------|----------------------------|----------------|-------
00000601-0000-1000-0000-009132591325 | Certificate Management Control Point | Write, Indicate            | no             |
00000602-0000-1000-0000-009132591325 | Certificate Management Data          | Write w/o response, Notify | no             |


## Firmware Update

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00000800-0000-1000-0000-009132591325 | Primary Service |

A Medtronic custom service.

### Service characteristics

UUID                                 | Name                          | Properties                 | SAKE-encrypted | Notes
-------------------------------------|-------------------------------|----------------------------|----------------|-------
00000801-0000-1000-0000-009132591325 | Firmware Update Control Point | Write, Indicate            | ?              |
00000802-0000-1000-0000-009132591325 | Firmware Update Data          | Write w/o response, Notify | ?              |
