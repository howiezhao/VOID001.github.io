---
title: Kernel Driver btusb Overview
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [November 2, 2017](https://web.archive.org/web/20210225082831/https://void-shana.moe/linux/kernel-driver-btusb-overview.html "1:59 pm") 
[VOID001](https://web.archive.org/web/20210225082831/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210225082831/https://void-shana.moe/linux/kernel-driver-btusb-overview.html#respond)





Function


### btusb\_probe


btusb\_probe is use for hot plug-in for bluetooth usb generic controller, here will explain the function in detail. 


First is an interface check mechanism


This special condition is used for supporting apple Macbook 12,8 (2015 early). According to the normal specification, the main interface for USB is 0, and audio (isochronous) is 1, but apple made a change on it, changing the main interface to 2 and audio to 3. The “**bInterfaceNumber !=2** ” is for checking hardware for the special case in Apple series product. The macro **BTUSB\_IFNUM\_2** is a *driver\_info* flag, for Macbook devices, this flag will be set, else it will be 0. See the **btusb\_table** for detail.


Then do further check on blacklist devices, some of the blacklist device is because there are specific driver (e.g bcmxxxx) for the device, so they do not use the generic one called btusb. Some of them just because they are not supported, and other reasons.(Not sure what reason are there)


 


Then we allocate memory for structure btusb\_data, use this to store data for the USB interface. Also we need to check the memory remained for the allocation. Then we do the real work: set up currrent interface endpoints for interrupt and bulk **(Why only these two?)** It go through all the endpoint in the current interface. We get the current\_altsetting to get a list of current active(available) endpoints.


 


**usb\_endpoint\_is\_int\_in** and **usb\_endpoint\_is\_bulk\_out**, **usb\_endpoint\_is\_bulk\_in** are functions use to know what type of the endpoint is it. These info is use to set up driver data at the end of the call. If none of inter\_ep, bulk\_tx\_ep or bulk\_rx\_ep is set, it will also result in No Device Error(**ENODEV**) 


This part of code is used for URB generation. URB is short for “USB Request Block” According to the Bluetooth v5.0 Specification, When sending an Control URB to AMP, the bRequest field should be 0x2b. Shown in the figure below.


 


Currently, for the interface to work with kernel to perform different operations. The driver itself need to be convert to device structure. Use the function named **interface\_to\_usbdev** Here is a quote from Linux Device Driver 3 :


*A USB device driver commonly has to convert data from a given* *struct* *usb\_interface* *structure into a* *struct* *usb\_device* *structure that the USB core needs for a wide range of function calls. To do this, the function interface\_to\_usbdev is provided. Hopefully, in the future, all USB calls that currently need a* *struct* *usb\_device* *will be converted to take a* *struct* *usb\_interface* *parameter and will not require the drivers to do the conversion.*


 


Then we continue with the initialize process.


 


Here we init the workqueue, **data->work** and **data->waker** these are shared workqueue offered by kernel. (Default Shared workqueue). We call **schedule\_work(data->work)** in **btusb\_notify** function to submit a job into workqueue and **data->wake**r is also controlled by other functions


Then these init\_usb\_anchor calls. In my view, is just a sort of data queue, URB request will be queued(anchored) in certain queue, then processed in serial. Then init the spinlock for the device(interface)


 


Another special case, for Intel bluetooth usb generic driver, kernel will use special recv handler functions, for other USB generic bluetooth driver, kernel just use the common one.


Then do a lot of device specific set-up, we skip the code and go to the  isochronous setup process.


 


 


Here, the **usb\_driver\_claim\_interface** is used for set up more than one interface binding to the current device driver. It also happens when this is a isochronous or acm(?) interface, here it’s a isochronous interface


Finally we call **hci\_register\_dev to register** it , this is one of the function in the Bluetooth Host Controller Interface core function series, from file net/bluetooth/hci/hci\_core.c. After that, we set the interface data to intf






---


[C](https://web.archive.org/web/20210225082831/https://void-shana.moe/category/linux/c), [Kernel](https://web.archive.org/web/20210225082831/https://void-shana.moe/category/kernel), [Linux](https://web.archive.org/web/20210225082831/https://void-shana.moe/category/linux) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
Kernel Bootup Page Table Initialize Process(x86\_64)](https://web.archive.org/web/20210225082831/https://void-shana.moe/linux/kernel-bootup-page-table-initialize-processx86_64.html)
[PREVIOUS 
Building your own live streaming site using Nginx RTMP & video.js](https://web.archive.org/web/20210225082831/https://void-shana.moe/linux/building-your-own-live-streaming-site-using-nginx-rtmp-video-js.html)

            