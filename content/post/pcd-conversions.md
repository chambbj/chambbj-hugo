+++
date = "2016-04-04T15:19:04-04:00"
draft = true
title = "PCD Conversions"

+++

While PDAL and PCL are both quite flexible in their point cloud representations, we are faced with a number of issues. First, PDAL maintains the notion of a [PointLayout](http://www.pdal.io/tutorial/overview.html#point-layout) that describes the various `Dimensions` that describe a point, and the associated datatype and name. PCL takes a different approach. It is a templated library, with the various point [types](http://pointclouds.org/documentation/tutorials/adding_custom_ptype.php#adding-custom-ptype), which can be predefined or custom to an application. Needless to say, there are no existing PCL point types for common layouts (like predefined LAS point record formats), and for self-describing formats like BPF, unless you know exactly what you're working with when writing the code, you'd pretty much be stuck with XYZ coordinates.

With all this variability, our current PCD drivers make some assumptions. The assumption is that many of the source data (to be converted to PCD) will be LAS, which contains (among other dimensions) XYZ and intensity, with the possibility of RGBA values. This should be enough information for many of the PCL algorithms and tools to produce meaningful results. There is also a very real possibility that the point coordinates will be in UTM. Upon converting to a float (for PCD), we will begin to see artifacts in the data. We can avoid this by offsetting (and perhaps scaling) the data. The default behavior is to use the minimum value in X, Y, and Z as the offsets for the respective dimensions (we do not scale). Finally, should intensity or RGBA values not be present in the input `PointLayout`, they will be zeros in the PCD file.

There is ongoing work to support PCD writer [upgrades](https://github.com/PDAL/PDAL/issues/1206), which should support user-defined output dimensions, along with user-defined offsets and scales.
