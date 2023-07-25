# smartpqi-bsd
**Microchip PQI FreeBSD Driver**

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
