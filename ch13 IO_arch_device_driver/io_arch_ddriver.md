# Chapter 13. I/O Architecture and Device Drivers
+ focus on: how the kernel invokes the operations on actual devices.

##13.1. I/O Architecture
+ discuss the functional characteristics common to all PC architectures

###13.1.1. I/O Ports
+ An important objective for system designers is to offer a unified approach to I/O programming without sacrificing performance.

####13.1.1.1. Accessing I/O ports
+ the kernel keeps track of I/O ports assigned to each hardware device by means of "resources ."
+ A *resource* represents a portion of some entity that can be exclusively assigned to a device driver. ---> 资源代表设备驱动独占的实体的分配
+ All resources of the same kind are inserted in a *tree-like* data structure;

####13.1.2. I/O Interfaces
+ An *I/O interface* is a hardware circuit inserted between a group of I/O ports and the corresponding device controller.
+ There are two types of interfaces:

| type                           | note                                               |
| ------------------------------ | -------------------------------------------------- |
| Custom I/O interfaces          | Devoted to one specific hardware device            |
| General-purpose I/O interfaces | Used to connect several different hardware devices |

####13.1.2.1. Custom I/O interfaces

| if                  | note                                                                        |
| ------------------- | --------------------------------------------------------------------------- |
| Keyboard interface  | Connected to a keyboard controller that includes a dedicated microprocessor |
| Graphic interface   | Packed together with the corresponding controller in a graphic card         |
| Disk interface      | Connected by a cable to the disk controller                                 |
| Bus mouse interface | Connected by a cable to the corresponding controller                        |
| Network interface   | Packed together with the corresponding controller in a network              |

####13.1.2.2. General-purpose I/O interfaces

| if                                               | note                                                                 |
| ------------------------------------------------ | -------------------------------------------------------------------- |
| Parallel port                                    | transferred 1 byte (8 bits) at a time                                |
| Serial port                                      | data is transferred 1 bit at a time                                  |
| PCMCIA interface                                 | used to connect external devices that do not operate at a high speed |
| SCSI (Small Computer System Interface) interface | hard disks, modems, network cards, and RAM expansions                |
| Universal serial bus (USB)                       | connects the main PC bus to a secondary bus                          |

###13.1.3. Device Controllers
+ the controller plays two important roles:
  + It interprets the high-level commands received from the I/O interface and forces the device to execute specific actions by sending proper sequences of electrical signals to it.
  + It converts and properly interprets the electrical signals received from the device and modifies (through the I/O interface) the value of the status register.

+ Simpler devices do not have a device controller. include PIC and PIT

##13.2. The Device Driver Model

###13.2.1. The sysfs Filesystem
+ A goal of the sysfs filesystem is to expose the hierarchical relationships among the components of the device driver model.
+ The main role of regular files in the sysfs filesystem is to represent attributes of drivers and devices.

###13.2.2. Kobjects
+ The core data structure of the device driver model is a generic data structure named kobject

####13.2.2.1. Kobjects, ksets, and subsystems
+ The kobjects can be organized in a hierarchical tree by means of ksets . A kset is a collection of kobjects of the *same type*.
+ a kset is a collection of kobjects, but it *relies on* a kobject of higher level for reference counting and linking in the hierarchical tree.

![](ddm_hierarchy.jpg)

####13.2.2.2. Registering kobjects, ksets, and subsystems
+  kobject directories include regular files called attributes

###13.2.3. Components of the Device Driver Model

####13.2.3.1. Devices
+  the structure of the directories below /sys/devices matches the physical organization of the hardware devices.
+  The *probe* method is invoked whenever a bus device driver discovers a device that could possibly be handled by the driver;

####13.2.3.3. Buses
+ Each bus_type object includes an embedded subsystem;
+ the devices directory of the per-bus subsystem stores symbolic links pointing to directories under /sys/devices. ---> /sys/bus/per_type/devices软链接到/sys/devices/per_type
+ The *match* method is executed when the kernel must check whether a given device can be handled by a given driver. Even if each device's identifier has a format specific to the bus that hosts the device, the function that implements the method is usually simple, because it searches the device's identifier in the driver's table of supported identifiers. ---> 在驱动表中检索是否存在对应标识，一般为“厂商ID + 设备ID”

####13.2.3.4. Classes
+ a logical device always refers to a given device in the device driver model.there can be several class_device descriptors that refer to the same device. ---> 多种类设备描述符指向同一个设备。

##13.3. Device Files
+ I/O devices are treated as special files called device files ;
+ device files can be of two types: block or character.
  + The data of a block device can be addressed randomly, and the time needed to transfer a data block is small and roughly the same
  + The data of a character device either cannot be addressed randomly (consider, for instance, a sound card), or they can be addressed randomly, but the time required to access a random datum largely depends on its position inside the device (consider, for instance, a magnetic tape driver).
+ Network cards are a notable **exception to this schema**, because they are hardware devices that are not directly associated with device files.
+  Its inode, however, doesn't need to include pointers to blocks of data on the disk (the file's data) because there are none. Instead, the inode must include an identifier of the hardware device corresponding to the character or block device file.
---> inode必须包含和字符或块设备文件相关的硬件ID
+ this identifier consists of the type of device file (character or block) and a pair of numbers.（设备类型 + 主设备号 +次设备号）
+ The *mknod( )* system call is used to create device files.

###13.3.1. User Mode Handling of Device Files
+ the size of the device numbers has been increased in Linux 2.6

####13.3.1.1. Dynamic device number assignment
+ Each device driver specifies in the registration phase the range of device numbers that it is going to handle ---> 每个驱动在注册阶段指定设备号范围。
+ the major and minor numbers are stored in the *dev* attributes contained in the subdirectories of /sys/class. ---> 主次设备号存储在dev属性文件中。

####13.3.1.2. Dynamic device file creation
+ At the system startup the /dev directory is emptied, then a *udev* program scans the subdirectories of /sys/class looking for the dev files.

###13.3.2. VFS Handling of Device Files
+  when a process accesses a device file, it is just driving a hardware device.

##13.4. Device Drivers
+ A device driver is the set of kernel routines that makes a hardware device respond to the programming interface defined by the canonical set of VFS functions (open, read, lseek, ioctl, and so forth) that control a device.

###13.4.1. Device Driver Registration
+ kernel relies on the *match* method of the relevant bus_type bus type descriptor, and on the *probe* method of the device_driver object.

###13.4.2. Device Driver Initialization
+ a device driver is initialized at the last possible moment. In fact, initializing a driver means allocating precious resources of the system, which are therefore not available to other drivers. ---> 尽可能晚的初始化设备驱动。资源只能独占。

###13.4.3. Monitoring I/O Operations
+ the device driver that started an I/O operation must rely on a monitoring technique that signals either the termination of the I/O operation or a time-out.
+ In the case of a terminated operation, the device driver reads the status register of the I/O interface to determine whether the I/O operation was carried out successfully. In the case of a time-out, the driver knows that something went wrong, because the maximum time interval allowed to complete the operation elapsed and nothing happened.

####13.4.3.1. Polling mode
+ the CPU checks (polls) the device's status register repeatedly until its value signals that the I/O operation has been completed.
+ If the time required to complete the I/O operation is relatively high, say in the order of milliseconds, this schema becomes inefficient because the CPU wastes precious machine cycles while waiting for the I/O operation to complete. ---> 完成IO操作的时间如果很长，将导致低效率（CPU浪费机器时间）。 it is preferable to voluntarily relinquish the CPU after each polling operation by inserting an invocation of the schedule( ) function inside the loop.

####13.4.3.2. Interrupt mode
+  be used only if the I/O controller is capable of signaling, via an IRQ line, the end of an I/O operation. ---> 通过中断线来发送IO操作结束信号。
+  This is a typical case in which it is preferable to implement the driver using the interrupt mode. Essentially, the driver includes two functions:
  +  The foo_read( ) function that implements the read method of the file object.
  +  The foo_interrupt( ) function that handles the interrupt.

###13.4.4. Accessing the I/O Shared Memory
+ Depending on the device and on the bus type, I/O shared memory in the PC's architecture may be mapped within different physical address ranges
  + For most devices connected to the ISA bus. mapped into the 16-bit physical addresses ranging from 0xa0000 to 0xfffff;
  + For devices connected to the PCI bus. mapped into 32-bit physical addresses near the 4 GB boundary.

+ I/O shared memory locations must be expressed as addresses greater than **PAGE_OFFSET**.(above 3G)

###13.4.5. Direct Memory Access (DMA)
+ With more modern bus architectures such as PCI, each peripheral can act as bus master.
+ The DMA is mostly used by disk drivers and other devices that transfer a large number of bytes at once.

####13.4.5.1. Synchronous and asynchronous DMA
+ A device driver can use the DMA in two different ways called synchronous DMA and asynchronous DMA. In the first case, the data transfers are triggered by processes; in the second case the data transfers are triggered by hardware devices.
  + An example of synchronous DMA is a sound card that is playing a sound track.
  + An example of asynchronous DMA is a network card that is receiving a frame (data packet) from a LAN.

####13.4.5.2. Helper functions for DMA transfers
+ There are two subsets of DMA helper functions: old + new

####13.4.5.3. Bus addresses
+ corresponds to the memory addresses used by all hardware devices except the CPU to drive the data bus. ---用于除CPU之外的硬件设备来驱动数据总线。
+ the data bus is driven directly by the I/O device and the DMA circuit. Therefore, when the kernel sets up a DMA operation, it must write the bus address of the memory buffer involved in the proper I/O ports of the DMA or I/O device.
+ All I/O drivers that make use of DMAs must set up properly the IO-MMU before starting the data transfer.--->所有使用DMA的IO驱动在启动数据发送之前必须合理的设置IO-MMU（X86架构除外）

####13.4.5.4. Cache coherency
+ If the DMA accesses the physical RAM locations but the corresponding hardware cache lines have not yet been written to RAM, then the hardware device fetches the *old values* of the memory buffer.
+ the developer chooses between two different DMA mapping types :

| mapping type         | note                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------ |
| Coherent DMA mapping | there will be no cache coherency problems between the memory and the hardware device |
| Streaming DMA mapping| must take care of cache coherency problems (synchronization helper functions).       |

+ if the buffer is accessed in unpredictable ways by the CPU and the DMA processor, coherent DMA mapping is *mandatory*

####13.4.5.5. Helper functions for coherent DMA mappings

####13.4.5.6. Helper functions for streaming DMA mappings

###13.4.6. Levels of Kernel Support
+ No support at all
+ Minimal support
+ Extended support
+ The minimal support approach is used to handle external hardware devices connected to a general-purpose I/O interface.
+ Minimal support is preferable to extended support because it keeps the kernel size small.
+ Minimal support has a limited range of applications, because it cannot be used when the external device must interact heavily with internal kernel data structures.---> 当需要和内核交互的时候此方案不再适用。

##13.5. Character Device Drivers
