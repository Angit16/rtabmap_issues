
# RTAB-Map mapping Guideline

## Important parameters
The following parameters should be configured appropriately during the mapping phase.
* While playing bag files for mapping set `use_sim_time` to **true**.
### Mapping with RGBD camera(s) and a LIDAR
Inside the scope of rtabmap node:
* Set `subscribe_scan_cloud` to **true**.
* Set `subscribe_rgbd` to **true**.
* Set `rgbd_cameras` to the chosen number of cameras to be used for the mapping.
* remap `scan_cloud` to an appropriate topic which is responsible for publishing LIDAR point clouds outside `RTAB-Map`.
* remap `rgbd_image` to an appropriate topic which is responsible for publishing rgbd images outside `RTAB-Map`.
* Registration strategy:  
  * Only camera, set `Reg/Strategy` to 0.
  * only LIDAR, set `Reg/Strategy` to 1.
  * camera + LIDAR, set `Reg/Strategy` to 2.
Set `Rtabmap/DetectionRate` to 3 to achieve an update ideally on every 0.333 second.
 
Inside the scope of visualizer node `rtabmapviz` or `rviz`:
* Set the above parameters, except the number of cameras in the visualizer node as well.

## Robot motion strategy during mapping
The progress of the map must be viewed by the user while moving the robot. It is important to not continue moving robot too far without findinf a global loop closure. 
# RTAB-Map map data processing from .db files

## Merging two maps
`rtabmap-reprocess "<input1.db>;<input2.db>" <output.db>`
 
 ## Select Distinct map ids in a .db file

 `sqlite3 <input.db> "select distinct (map_id) from Node"`

 To  Select Max and Min node position of any map from 

`sqlite3 <input.db> "select distinct (map_id) from Node"`

 <!-- --- -->

 To remove a portion from an existing map,
 run `map_operate.sh`
 Enter the map file name by passing `$map_name` as an argument. In the file  `map_operate.sh`, nodes removal starts from `from_id`. Similarly, `to_id` refers to the stopping node ID to be removed. Therefore one has to specify the desired node IDs to the respecctive fields.

*Nodes removal Strategy:*

 ## Post processing a map from a .db file
`rtabmap-reprocess --<rtabmap argument> <input.db> <output.db>`

For example, **to ensure that all the submaps post the execution of** `map_operate.sh` are smaller than the original `input.db`, a sufficient command is 

`rtabmap-reprocess --Mem/LaserScanVoxelSize 0.1 <input1.db> <output.db>`
, which will shrink the `output.db` while downsampling the grid map with the specified voxel size.

Another way to shrink or rebuild a `.db` file is to use [VACUUM](https://www.sqlite.org/lang_vacuum.html).

**Important:** While removing a group of nodes from a .db file, make sure to not remove a few initial nodes from that map until the occurance of the first global loop closure. Otherwise, the map origin may shift to a different one than the original map's origin. For the OLA mapping activiteis, the entire environment has been divided into many small and consecutively overlapping regions. A map for each region has been created independently. All these maps are then merged together using the map merging procedure explained above. Now, the global reference is created with reference to a map that contained the first global loop closure during the merging process. So, while generating smaller maps from a large merged map, one needs to ensure the following points.

* The map containing the first global loop closure must be retained in the smaller map. Ideally, all the loop closures and their neighboring nodes should be retained in it.
* `rtabmap-reprocess` should be executed post that (`./map_operate.sh`) to reduce the map size.

## Map switching

Loading a map dynamically, while another map is already in use is termed as *Map Switching*. While, the new map may store a different last localized pose in its database compared to the robot's current pose, the robot's entry into the new map should not suffer from that and should be set to a near accurate *init_pose*. Finding one or more global loop closure(s) is critically important to initialize the localization within the new map. Therefore, choosing a map switching region somewhere close to such loop closures is effective towards localizing the robot with a greater accuracy even with a rough estimate of *init_pose* with the help of the odometry.

## Mapping Pipeline

The following flow diagram depicts the present mapping pipeline. 
