# orb_slam3_for_ros
This repo put [ORB_SLAM3](https://github.com/UZ-SLAMLab/ORB_SLAM3) in ROS with **catkin_make_isolated** and install option. *DBoW2* and *g2o* are treated as non-catkin packages.

Put all the directories into work space *src*, and run:

```
  catkin_make_isolated --install
```

Remark 1: `-j1` is needed if you are running in a virtual machine or other low performance devices.

Remark 2: `--use-ninja` can be used as well if you would like to integrate [Cartographer](https://google-cartographer-ros.readthedocs.io/en/latest/).




--------------------------------------------------