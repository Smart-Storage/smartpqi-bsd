# smartpqi-bsd
**Microchip PQI FreeBSD Driver**

### Version 4540.0.1005 (May 2024)

### Version 4530.0.1010 (April 2024)

### Version 4520.0.1015 (March 2024)

### Version 4510.0.1018 (February 2024)


### Version 4500.0.1024 (December 2023)

 - feat: Adjust CAM_BUSY

    During a device reset, only set the CAM_BUSY status flag.
    (Suggested by upstream maintainer during code review.)


### Version 4490.0.1013 (December 2023)

 - fix: With INVARIANTS enabled kernel, panic observed while creating & deleting array.

    Root cause: While creating & deleting memory was freeing
    inside the spinlock.

    Fix: Move the memory freeing outside the spinlock

    Risk: Low


 - fix: Panic observed when removing drive with INVARIANTS enabled

    Problem: With INVARIANTS enabled in the kernel , panic observed
    while hot-removing a drive

    Root cause: pqisrc_free_device function frees device memory
    inside the spinlock.

    Fix: Move the memory freeing outside the spinlock.

    Risk: Low

 - feat:  Add support for FreeBSD 14.0

### Version 4480.0.1021 (November 2023)

### Version 4470.0.1012 (November 2023)

 - feat:  Add logging levels as a tunable value

    Adds the ability to set the driver debugging
    levels in the loader.conf file.

### Version 4450.0.1009 (August 2023)

 - fix: Change alignment for dma tags

    Problem: Under certain I/O conditions, a program doing large block disk reads
    can cause a controller to crash.

    Root Cause: The SCSI read request and destination address in the BDMA descriptor
    is incorrect, causing the BDMA engine in the controller to assert.

    Fix: Change the alignment for creating bus_dma_tags in the driver from
    PAGE_SIZE (4k) to 1, which allows the controller to manage it's own address
    range for BDMA transactions.

    Risk: Medium

    Exposure: Since 4034.0

### Version 4410.0.1005 (July 2023)
 - Added support for FreeBSD 13.2

### Version 4390.0.1010 (March 2023)
 - fix: Wrong LUN is getting reset when LUN RESET TMF issued

    Problem: While issuing LUN reset TMF to second LUN of
    dual-actuator drive, the first LUN is also reset.

    Root Cause: In the current code, lun-address & lun number
    fields are not updated in tmf req structure for multi-lun
    devices.

    Fix: Added changes in AIO/RAID TMF structures to send tmf
    request for multi lun devices.

    Risk: Low

 - fix: Partition information for all LUNs of a multi-actuator drives is reported as
    being the same after hot removing and then hot re-adding the drive

    Problem: For a multi-actuator drive, partition information for LUN1 is being reported
    with the information from LUN0 after a hot-removal & hot-reinsert of the drive.

    Root Cause: After hot-reinserting a multi-actuator drive, IO to LUNs is submitted
    using the RAID path. However, commands sent to multi-actuator drives through the RAID
    path are not handled correctly.

    Fix: Added support to send commands to multi-actuator drives through the RAID path when
    needed.

    Risk: medium

 - Added support for FreeBSD 12.4

    With FreeBSD 12.4 being released, we moved our builds
    from FreeBSD 12.3 to FreeBSD 12.4.

 - feat: Add support for multi-actuator hard drives

### Version 4330.0.1038 (November 2022)

### Version 4280.0.1007 (August 2022)
 - fix: Newly created LDs are not listing in geom disk list command

    Problem: Newly created LD are not listing on geom disk list
    command.

    Root cause: For newly created LDs ,os_rescan_target function
    is called before adding the device to cam layer.
    os_rescan_target function is used to notify os regarding the
    logical volume expansion.

    Fix: os_rescan_target function will only be called for
    existing logical volumes.

    Risk: Low

 - fix: Logical Devices are not listed with "geom disk list" command

    Problem: Logical Devics created on HDD drives were not being
    listed with the "geom disk list" command.

    Root Cause: The bus_dmamap_sync operation which is performed before
    device access was being handled incorrectly.

    Fix: Handle the bus_dmamap_sync operation correctly.

    Risk: Low

 - fix: Expanded LD device size is not reflecting in OS

    Problem: Expanded Logical device size is not
    reflecting in OS after modifying the LD size from
    arcconf.

    Root Cause: Current FreeBSD code detects the volume
    expansion , by checking the volume status of the device.

    FW just needs the time to process the BMIC and the time
    it takes for the volume state to change from
    OK->NEEDS_EXPAND->EXPANDING->OK will be a fraction of a
    second.
    So due to this reason, by the time driver sends the
    Inquiry command the volume expansion would have been
    completed and the volume state would have come back
    to the OK state. This results in driver to fail in
    detecting the volume expansion.

    Fix: Driver detects the event type
    PQI_EVENT_TYPE_LOGICAL_DEVICE (0x5), as result of
    logical device expansion & sets the rescan flag.
    It would later trigger reprobe  of all LDs, which
    includes sending SCSI READ CAPACITY command and
    updating the size which visible to OS.

    Risk: Low

 - fix: Correct data direction for Raidbypass IO

    Problem: OS commands are getting hung after
    executing on logical volumes created using SSD.

    Root Cause: Driver was not properly handling
    the data direction flag for raidbypass IO.

    Fix: Corrected the data direction flag for
    raidbypass request

    Risk: Low

 - fix: Correct data direction flag for the request

    Problem: SCSI READ BLOCK LIMITS (0x5) command is never
    completed by firmware and a TMF ABORT is observed.

    Root Cause: Driver is sending incorrect data direction
    flag.

    Fix: Corrected the data direction flag.

    Risk: Low

### Version 4210.0.1004 (February 2022)
 - feat: Change 32 bit DMA address to 64 bit

    Current FreeBSD driver creates 32 bit DMA address.
    This has to be changed to support 64 bit address.

 - fix: Resolves inconsistent sequential read performance with RAID10 volumes using AIO path

    Problem: Testing discovered inconsistent performance on RAID 10 volumes
    when performing 256K sequential reads.

    Root Cause: The driver was only using a single tracker to determine
    which physical drive to send a request to for AIO requests.

    Fix: Allocate an array of trackers based on the number of data disks in
    a row of the RAID map.

    Risk: Medium

    Exposure: All prior versions
