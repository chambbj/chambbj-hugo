+++
date = "2016-04-04T13:49:51-04:00"
draft = false
title = "Crossing the Streams"

+++

In January of this year, I was convinced to submit an abstract for [FOSS4G North America 2016](https://2016.foss4g-na.org/). To my delight, my talk on [Filtering point clouds with PDAL and PCL](https://2016.foss4g-na.org/session/filtering-point-clouds-pdal-and-pcl) was accepted. This is a topic that is near and dear to me as an early adopter and contributor to both [PDAL](http://pdal.io) and [PCL](http://pointclouds.org).

<!--more-->

To this day, I find it quite convenient to leverage PCL's extensive collection of [modules](http://docs.pointclouds.org/1.7.2/modules.html) when developing new approaches to processing point cloud data. Though the pace of PCL development has slowed (latest [release](https://github.com/PointCloudLibrary/pcl/releases/tag/pcl-1.7.2), [workshop](http://ns50.pointclouds.org/news/2014/04/03/pcl-tutorial-and-3drp-pcl-workshop-at-ias-2014/) and [code sprint](http://ns50.pointclouds.org/news/2014/02/12/new-ocular-robotics-pcl-code-sprint/) were all in 2014), there is still a wealth of algorithms that can aid in point cloud processing and analysis tasks. And it is easily [extensible](http://ns50.pointclouds.org/documentation/tutorials/writing_new_classes.php#writing-new-classes).

While point clouds can be derived from a [number](http://www.wired.com/2014/09/velodyne-lidar-self-driving-cars/) of [sources](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=4359315&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D4359315), my focus continues to be on point clouds collected by airborne lidar systems, where [LAS](http://www.asprs.org/Committee-General/LASer-LAS-File-Format-Exchange-Activities.html), [LAZ](http://www.laszip.org/), and the lesser known [BPF](https://nsgreg.nga.mil/doc/view?i=4202) formats are the norm. When it comes to reading/writing these (and other) formats, I'd rather not worry about the details, which is were PDAL shines. The PDAL [CLI](http://www.pdal.io/apps.html) allows me to effortlessly transcode between formats using the [translate](http://www.pdal.io/apps.html#translate-command) subcommand.

```console
$ pdal translate -i input.bpf -o output.laz
```

Here we have converted a point cloud input written in the BPF format to a point cloud compressed as LAZ. But the fun doesn't end there! We can also construct processing pipelines by inserting [filters](http://www.pdal.io/stages/index.html#filters).

```console
$ pdal translate -i input.bpf -o output.laz -f range \
    --filters.range.limits="Z(0:]"
```

This example performs the same format conversion, but uses a [range](http://www.pdal.io/stages/filters.range.html) filter to only pass points with Z values that are greater than 0.

Pipelines can also be specified as JSON, invoked using the [pipeline](http://www.pdal.io/apps.html#pipeline-command) subcommand.

```console
$ pdal pipeline -i pipeline.json
```

Here is the earlier transcoding example, specified using PDAL's JSON [specification](http://www.pdal.io/json_pipeline_specification.html#json-pipeline-specification).

```json
{
  "pipeline":[
    "input.bpf",
    "output.laz"
  ]
}
```

Similarly, the range filtering example is given by:

```json
{
  "pipeline":[
    "input.bpf",
    {
      "type":"filters.range",
      "limits":"Z(0:]"
    },
    "output.laz"
  ]
}
```

This barely scratches the surface of what PDAL can do, but I think you get the idea.

While it was at first tempting to either 1) write format drivers for PCL or 2) write the processing algorithms for PDAL, both of these overlook a vital aspect of open source software: **community**. If I were to go with option 1, I'd be on an island -- at least initially. The established PCL community already had a [format](http://pointclouds.org/documentation/tutorials/pcd_file_format.php). The same was true for option 2. PDAL's goal in life is really to focus on formats. Sure it would be nice to have many of the PCL algorithms living natively within PDAL, but I don't want to spend the bulk of my time recoding a bunch of algorithms, and it didn't seem there was a body of developers who wanted to jump in on the task with me.

No. What I really want to do is to come up with novel ways of processing the data. Sometimes this will mean writing (or implementing) a [new](http://www.pdal.io/tutorial/calculating-normalized-heights.html) or [existing](http://www.pdal.io/tutorial/pcl_ground.html) algorithm. Other times it's simply a matter of wiring up existing algorithms in a [new way](http://www.pdal.io/json_pipeline_specification.html#extended-examples). To that end, we've developed methods for incorporating PCL within PDAL, along with new and intuitive ways to interact with PDAL. Over the coming weeks, my hope is to be able to share with you a number of methods we have developed that bridge the PDAL-PCL divide.
