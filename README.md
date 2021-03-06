# uvc-gadget

UVC gadget test application Readme

Copyright (C) 2010 Ideas on board SPRL

Copyright (C) 2013 ST Microelectronics Ltd.

Copyright (C) 2016 Flextronics International

Contacts:
Laurent Pinchart <laurent.pinchart@xxxxxxxxxxxxxxxx>

Bhupesh Sharma <bhupesh.sharma@xxxxxx>

Frank Earl <linusti@xxxxxxxxx>


Introduction
============

This README file documents the UVC gadget test application available
here git://git.ideasonboard.org/uvc-gadget.git, which can be used to
test the UVC webcam gadget driver located under drivers/usb/gadget
directory.

Possible Use-Case scenarios
===========================

There can be a number of scenarios under which the UVC webcam gadget can
be used and each use-case can vary in the following ways:

- Video streaming endpoint choice:
  --------------------------------

  As per UVC specifications (Revision 1.1), either Bulk or Isochronous
  USB endpoints can be used are for implementing UVC Video streaming
  interface. Both have there unique challenges as:

	Isochronous video streaming interface will have two
	alt-settings:
		- Alt-setting 0 for zero-bandwidth mode, and
		- Alt-setting 1 for full-bandwidth mode.

	This requires us to ACK status stage of SET_ALT(alt-setting =
	1), only when we have video frames available which can be queued
	to UVC domain.

	Bulk video streaming interface, on the other hand will have only
	one alt-setting.

	This requires us to prepare UVC gadget for streaming when we
	receive a a UVC_VS_COMMIT_CONTROL command from USB host and
	actually start streaming as soon as we have video frames
	available which can be queued to UVC domain.

- Availability of a video capture device:
  ---------------------------------------

  To emulate a real UVC webcam gadget (capturing live video from a video
  capture source and sending it over USB bus), it is possible to
  integrate a V4L2 based video-capture device driver with the UVC gadget
  test application acting as an interface between the UVC based
  video-output device and V4L2 based video-capture device.
  
  Accordingly, there can be the following scenarios possible:

	- No video capture device available:

		In this case, there is no video capture device available
		and the UVC webcam gadget is working in an standalone
		environment.

		It is possible in this case to stream a dummy test
		pattern from the UVC webcam gadget and view the same on
		the Linux/Windows USB Host machine using standard video
		capture utilities like CHEESE.

	- VIVI (Virtual Video driver) acting as a video capture source:

		In case we don't have a real video capture hardware
		(e.g. sensor) available, it is still possible to
		exercise the complete path from the V4L2 based
		video-capture domain to UVC based video-output domain,
		using the Virtual Video driver (VIVI) available here:
		drivers/media/platform/vivi.c

		It is possible in this case to stream changing vertical
		color bar pattern pattern from the VIVI capture device
		and send it to UVC webcam gadget, which will eventually
		send these frames over USB bus.

		Similar to the case above, it is possible to view the
		frames on the Linux/Windows USB Host machine using
		standard video capture utilities like CHEESE.

	- Real video capture hardware and corresponding V4L2 driver
	  available:

		In this case, we have a real video capture hardware
		(e.g. CCD / CMOS sensor) available.

		Here, it is possible to exercise the complete path from
		the V4L2 based video-capture driver domain to UVC based
		video-output domain, using the UVC gadget test
		application.

		It is possible in this case to stream real video/still
		images from capture source and send it to UVC webcam
		gadget, which will eventually send these frames over USB
		bus.

- IO methods supported:
  --------------------

  V4L2 framework supports a number of IO methods and to achieve
  zero-memcpy of video frames as they pass from V4L2 video-capture
  domain to UVC video-output domain, it is important to select
  appropriate IO methods for working on videobuffers.

  Accordingly we can have the following scenarios possible:
  
	- UVC webcam standalone, supporting IO_METHOD_MMAP method:

		In this case, there is no video capture device available
		and the UVC webcam gadget is working in an standalone
		environment.

		It is supporting IO_MMAP method, which implies that the
		videobuffers are allocated by the UVC webcam gadget
		itself and they are filled with dummy test pattern by
		UVC gadget test application.

	- UVC webcam standalone, supporting IO_METHOD_USERPTR method:

		In this case, there is no video capture device available
		and the UVC webcam gadget is working in an standalone
		environment.

		It is supporting IO_METHOD_USERPTR method, which implies
		that the videobuffers are allocated and filled with
		dummy test pattern by the UVC gadget test application
		and UVC webcam gadget operates on a USERPTR of these
		videobuffers.

	- UVC webcam integrated with V4L2 video-capture driver, with
	  V4L2 video-capture driver supporting IO_METHOD_MMAP method and
	  UVC webcam supporting IO_METHOD_USERPTR:

		In this case, either VIVI or another V4L2 based driver
		(working on real HW) is available as a video capture
		device and is integrated with the UVC webcam gadget via
		UVC gadget test application, with V4L2 video-capture
		driver supporting IO_METHOD_MMAP method and UVC webcam
		supporting IO_METHOD_USERPTR.

		Here, the videobuffers are allocated in the V4L2
		video-capture domain and UVC gadget test application
		acts as a means to pass a USERPTR of these videobuffers
		to the UVC webcam gadget domain, without requiring any
		memcpy from the CPU.

	- UVC webcam integrated with V4L2 video-capture driver, with
	  V4L2 video-capture driver supporting IO_METHOD_USERPTR method
	  and UVC webcam supporting IO_METHOD_MMAP:

		In this case, either VIVI or another V4L2 based driver
		(working on real HW) is available as a video capture
		device and is integrated with the UVC webcam gadget via
		UVC gadget test application, with V4L2 video-capture
		driver supporting IO_METHOD_USERPTR method and UVC
		webcam supporting IO_METHOD_MMAP.

		Here, the videobuffers are allocated in the UVC webcam
		gadget domain and UVC gadget test application acts as a
		means to pass a USERPTR of these videobuffers to the
		V4L2 video-capture domain, without requiring any memcpy
		from the CPU.

Modifying UVC gadget test application
=====================================

The UVC webcam gadget kernel driver located under drivers/usb/gadget
directory and compilable as g_webcam.ko kernel module, supports 360p and
720p frame resolutions and YUV4:2:2 and MJPEG frame formats by default.

So, the video capture device must be aligned to support similar frame
format and resolutions.

In case the video capture device supports other resolutions and frame
formats, we need to replace the default values used in the UVC gadget
test application with the same and also update the UVC webcam gadget
kernel driver.

Keeping USBCV's UVC test-suite happy
====================================

The UVC webcam gadget kernel driver located under drivers/usb/gadget
directory and compilable as g_webcam.ko kernel module, supports changing
the Brightness attribute of the Processing Unit (PU) by default.

If the underlying video capture device supports changing the Brightness
attribute of the image being acquired (like the Virtual Video, VIVI
driver does), then we should route this UVC request to the respective
video capture device.

Incase, there is no actual video capture device associated with the UVC
gadget and we wish to use this application as the final destination of
the UVC specific requests then we should return pre-cooked (static)
responses to GET_CUR(BRIGHTNESS) and SET_CUR(BRIGHTNESS) commands to
keep command verifier test tools like UVC class specific test suite of
USBCV, happy.

Note that the values used in UVC gadget test application are in sync
with the VIVI driver and must be changed for your specific video capture
device. These values also work well in case there in no actual video
capture device.

