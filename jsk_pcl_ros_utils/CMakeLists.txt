# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(jsk_pcl_ros_utils)

if($ENV{ROS_DISTRO} STREQUAL "groovy")
  # update package.xml, in groovy we need to add pcl to package.xml
  execute_process(COMMAND sed -i s@<run_depend>pcl_ros</run_depend>@<run_depend>pcl_ros</run_depend><run_depend>pcl</run_depend>@g ${PROJECT_SOURCE_DIR}/package.xml)
  execute_process(COMMAND sed -i s@<build_depend>pcl_ros</build_depend>@<build_depend>pcl_ros</build_depend><build_depend>pcl</build_depend>@g ${PROJECT_SOURCE_DIR}/package.xml)
endif($ENV{ROS_DISTRO} STREQUAL "groovy")


# Load catkin and all dependencies required for this package
# TODO: remove all from COMPONENTS that are not catkin packages.
if($ENV{ROS_DISTRO} STREQUAL "groovy")
  set(PCL_MSGS pcl)
else()
  set(PCL_MSGS pcl_msgs) ## hydro and later
endif()

if($ENV{ROS_DISTRO} STREQUAL "groovy")
  set(ML_CLASSIFIERS )
else()
  set(ML_CLASSIFIERS ml_classifiers) ## hydro and later
endif()

find_package(catkin REQUIRED COMPONENTS
  jsk_recognition_utils
  dynamic_reconfigure pcl_ros nodelet message_generation genmsg
  ${PCL_MSGS} sensor_msgs geometry_msgs jsk_recognition_msgs
  eigen_conversions tf_conversions tf2_ros tf
  image_transport nodelet cv_bridge
  sklearn jsk_topic_tools
  image_geometry
  jsk_footstep_msgs
  interactive_markers
  laser_assembler
  octomap_server
  octomap_ros
  octomap_msgs
  kdl_parser
  kdl_conversions
  )
find_package(PkgConfig)
pkg_check_modules(${ML_CLASSIFIERS} ml_classifiers QUIET)
# only run in hydro
if(NOT $ENV{ROS_DISTRO} STREQUAL "groovy")
  find_package(PCL REQUIRED)
endif(NOT $ENV{ROS_DISTRO} STREQUAL "groovy")
find_package(OpenMP)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")

# For kinfu
find_package(PCL QUIET COMPONENTS gpu_kinfu_large_scale)

pkg_check_modules(yaml_cpp yaml-cpp REQUIRED)
IF(${yaml_cpp_VERSION} VERSION_LESS "0.5.0")
## indigo yaml-cpp : 0.5.0 /  hydro yaml-cpp : 0.3.0
  add_definitions("-DUSE_OLD_YAML")
ENDIF()

# add_service_files(FILES SwitchTopic.srv
#   UpdateOffset.srv
#   SnapFootstep.srv
#   SetDepthCalibrationParameter.srv
#   TransformScreenpoint.srv
#   CheckCircle.srv
#   RobotPickupReleasePoint.srv
#   TowerPickUp.srv
#   EuclideanSegment.srv
#   TowerRobotMoveCommand.srv
#   SetPointCloud2.srv
#   CallSnapIt.srv CallPolygon.srv
#   EnvironmentLock.srv
#   PolygonOnEnvironment.srv
#   ICPAlignWithBox.srv
#   ICPAlign.srv
#   CheckCollision.srv)

# generate the dynamic_reconfigure config file
generate_dynamic_reconfigure_options(
#   cfg/VoxelGridLargeScale.cfg
  cfg/PolygonArrayAreaLikelihood.cfg
#   cfg/PlaneSupportedCuboidEstimator.cfg
#   cfg/InteractiveCuboidLikelihood.cfg
#   cfg/ExtractParticlesTopNBase.cfg
  cfg/PoseWithCovarianceStampedToGaussianPointCloud.cfg
#   cfg/HeightmapMorphologicalFiltering.cfg
#   cfg/HeightmapConverter.cfg
#   cfg/HeightmapToPointCloud.cfg
#   cfg/HeightmapTimeAccumulation.cfg
  cfg/PolygonMagnifier.cfg
  cfg/PolygonPointsSampler.cfg
#   cfg/GeometricConsistencyGrouping.cfg
#   cfg/UniformSampling.cfg
  cfg/PlanarPointCloudSimulator.cfg
  cfg/SphericalPointCloudSimulator.cfg
#   cfg/BorderEstimator.cfg
#   cfg/HintedStickFinder.cfg
#   cfg/HintedPlaneDetector.cfg
#   cfg/TorusFinder.cfg
  cfg/PlaneConcatenator.cfg
#   cfg/NormalDirectionFilter.cfg
#   cfg/RegionGrowingMultiplePlaneSegmentation.cfg
#   cfg/LineSegmentCollector.cfg
#   cfg/LineSegmentDetector.cfg
#   cfg/ParticleFilterTracking.cfg
#   cfg/BilateralFilter.cfg
#   cfg/ICPRegistration.cfg
  cfg/PlaneReasoner.cfg
#   cfg/OrganizedPassThrough.cfg
#   cfg/EuclideanClustering.cfg
  cfg/ColorizeDistanceFromPlane.cfg
#   cfg/HSIColorFilter.cfg
#   cfg/RGBColorFilter.cfg
#   cfg/ImageRotate.cfg
#   cfg/RegionGrowingSegmentation.cfg
#   cfg/OrganizedMultiPlaneSegmentation.cfg
#   cfg/MultiPlaneExtraction.cfg
#   cfg/NormalEstimationIntegralImage.cfg
  cfg/PlaneRejector.cfg
#   cfg/EnvironmentPlaneModeling.cfg
#   cfg/ColorHistogramMatcher.cfg
#   cfg/GridSampler.cfg
#   cfg/OrganizedEdgeDetector.cfg
#   cfg/EdgeDepthRefinement.cfg
#   cfg/ParallelEdgeFinder.cfg
#   cfg/EdgebasedCubeFinder.cfg
#   cfg/MultiPlaneSACSegmentation.cfg
#   cfg/BoundingBoxFilter.cfg
  cfg/MaskImageToDepthConsideredMaskImage.cfg
#   cfg/ResizePointsPublisher.cfg
#   cfg/LINEMODDetector.cfg
#   cfg/SupervoxelSegmentation.cfg
#   cfg/FeatureRegistration.cfg
#   cfg/FisheyeSphere.cfg
#   cfg/MovingLeastSquareSmoothing.cfg
#   cfg/OctreeVoxelGrid.cfg
#   cfg/OctreeChangePublisher.cfg
  )

generate_messages(DEPENDENCIES
  ${PCL_MSGS} sensor_msgs geometry_msgs jsk_recognition_msgs jsk_footstep_msgs)
find_package(OpenCV REQUIRED core imgproc)

include_directories(include ${catkin_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS} ${PCL_INCLUDE_DIRS})
link_directories(${catkin_LIBRARY_DIRS})
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -z defs")

macro(jsk_pcl_util_nodelet _nodelet_cpp _nodelet_class _single_nodelet_exec_name)
  message("util: ${_nodelet_cpp}")
  jsk_nodelet(${_nodelet_cpp} ${_nodelet_class} ${_single_nodelet_exec_name}
    jsk_pcl_util_nodelet_sources jsk_pcl_util_nodelet_executables)
endmacro(jsk_pcl_util_nodelet _nodelet_cpp _nodelet_class _single_nodelet_exec_name)

if ($ENV{TRAVIS_JOB_ID})
  add_definitions("-O0")
else ($ENV{TRAVIS_JOB_ID})
  add_definitions("-O2 -g")
endif ($ENV{TRAVIS_JOB_ID})

jsk_pcl_util_nodelet(src/normal_flip_to_frame_nodelet.cpp "jsk_pcl/NormalFlipToFrame"
  "normal_flip_to_frame")
jsk_pcl_util_nodelet(src/centroid_publisher_nodelet.cpp "jsk_pcl/CentroidPublisher" "centroid_publisher")
jsk_pcl_util_nodelet(src/tf_transform_cloud_nodelet.cpp
  "jsk_pcl/TfTransformCloud" "tf_transform_cloud")
jsk_pcl_util_nodelet(src/tf_transform_bounding_box_nodelet.cpp
  "jsk_pcl/TfTransformBoundingBox" "tf_transform_bounding_box")
jsk_pcl_util_nodelet(src/tf_transform_bounding_box_array_nodelet.cpp
  "jsk_pcl/TfTransformBoundingBoxArray" "tf_transform_bounding_box_array")
jsk_pcl_util_nodelet(src/normal_concatenater_nodelet.cpp
  "jsk_pcl/NormalConcatenater" "normal_concatenater")
jsk_pcl_util_nodelet(src/polygon_array_distance_likelihood_nodelet.cpp
  "jsk_pcl/PolygonArrayDistanceLikelihood" "polygon_array_distance_likelihood")
jsk_pcl_util_nodelet(src/polygon_array_area_likelihood_nodelet.cpp
  "jsk_pcl/PolygonArrayAreaLikelihood" "polygon_array_area_likelihood")
jsk_pcl_util_nodelet(src/polygon_array_angle_likelihood_nodelet.cpp
  "jsk_pcl/PolygonArrayAngleLikelihood" "polygon_array_angle_likelihood")
jsk_pcl_util_nodelet(src/polygon_array_foot_angle_likelihood_nodelet.cpp
  "jsk_pcl/PolygonArrayFootAngleLikelihood" "polygon_array_foot_angle_likelihood")
jsk_pcl_util_nodelet(src/pose_with_covariance_stamped_to_gaussian_pointcloud_nodelet.cpp
  "jsk_pcl/PoseWithCovarianceStampedToGaussianPointCloud" "pose_with_covariance_stamped_to_gaussian_pointcloud")
jsk_pcl_util_nodelet(src/spherical_pointcloud_simulator_nodelet.cpp
  "jsk_pcl/SphericalPointCloudSimulator" "spherical_pointcloud_simulator")
jsk_pcl_util_nodelet(src/polygon_flipper_nodelet.cpp
  "jsk_pcl/PolygonFlipper" "polygon_flipper")
jsk_pcl_util_nodelet(src/polygon_points_sampler_nodelet.cpp
  "jsk_pcl/PolygonPointsSampler" "polygon_points_sampler")
jsk_pcl_util_nodelet(src/polygon_magnifier_nodelet.cpp
  "jsk_pcl/PolygonMagnifier" "polygon_magnifier")
jsk_pcl_util_nodelet(src/planar_pointcloud_simulator_nodelet.cpp
  "jsk_pcl/PlanarPointCloudSimulator" "planar_pointcloud_simulator")
jsk_pcl_util_nodelet(src/plane_rejector_nodelet.cpp
  "jsk_pcl/PlaneRejector" "plane_rejector")
jsk_pcl_util_nodelet(src/pointcloud_to_cluster_point_indices_nodelet.cpp
  "jsk_pcl/PointCloudToClusterPointIndices" "pointcloud_to_cluster_point_indices")
jsk_pcl_util_nodelet(src/static_polygon_array_publisher_nodelet.cpp
  "jsk_pcl/StaticPolygonArrayPublisher" "static_polygon_array_publisher")
jsk_pcl_util_nodelet(src/polygon_array_transformer_nodelet.cpp
  "jsk_pcl/PolygonArrayTransformer" "polygon_array_transformer")
jsk_pcl_util_nodelet(src/pointcloud_to_stl_nodelet.cpp
  "jsk_pcl/PointCloudToSTL" "pointcloud_to_stl")
jsk_pcl_util_nodelet(src/polygon_appender_nodelet.cpp
  "jsk_pcl/PolygonAppender" "polygon_appender")
jsk_pcl_util_nodelet(src/delay_pointcloud_nodelet.cpp
  "jsk_pcl/DelayPointCloud" "delay_pointcloud")
jsk_pcl_util_nodelet(src/depth_image_error_nodelet.cpp
  "jsk_pcl/DepthImageError" "depth_image_error")
jsk_pcl_util_nodelet(src/polygon_array_wrapper_nodelet.cpp
  "jsk_pcl/PolygonArrayWrapper" "polygon_array_wrapper")
jsk_pcl_util_nodelet(src/polygon_array_unwrapper_nodelet.cpp
  "jsk_pcl/PolygonArrayUnwrapper" "polygon_array_unwrapper")
jsk_pcl_util_nodelet(src/colorize_distance_from_plane_nodelet.cpp
  "jsk_pcl/ColorizeDistanceFromPlane" "colorize_distance_from_plane")
jsk_pcl_util_nodelet(src/colorize_height_2d_mapping_nodelet.cpp
  "jsk_pcl/ColorizeHeight2DMapping" "colorize_height_2d_mapping")
jsk_pcl_util_nodelet(src/plane_reasoner_nodelet.cpp
  "jsk_pcl/PlaneReasoner" "plane_reasoner")
jsk_pcl_util_nodelet(src/transform_pointcloud_in_bounding_box_nodelet.cpp
  "jsk_pcl/TransformPointcloudInBoundingBox" "transform_pointcloud_in_bounding_box")
jsk_pcl_util_nodelet(src/point_indices_to_mask_image_nodelet.cpp
  "jsk_pcl/PointIndicesToMaskImage" "point_indices_to_mask_image")
jsk_pcl_util_nodelet(src/mask_image_to_depth_considered_mask_image_nodelet.cpp
  "jsk_pcl/MaskImageToDepthConsideredMaskImage" "mask_image_to_depth_considered_mask_image")
jsk_pcl_util_nodelet(src/mask_image_to_point_indices_nodelet.cpp
  "jsk_pcl/MaskImageToPointIndices" "mask_image_to_point_indices")
jsk_pcl_util_nodelet(src/plane_concatenator_nodelet.cpp
  "jsk_pcl/PlaneConcatenator" "plane_concatenator")
jsk_pcl_util_nodelet(src/add_point_indices_nodelet.cpp
  "jsk_pcl/AddPointIndices" "add_point_indices")
jsk_pcl_util_nodelet(src/pcd_reader_with_pose_nodelet.cpp
  "jsk_pcl/PCDReaderWithPose" "pcd_reader_with_pose")
add_library(jsk_pcl_ros_utils SHARED ${jsk_pcl_util_nodelet_sources})
add_dependencies(jsk_pcl_ros_utils ${PROJECT_NAME}_gencpp ${PROJECT_NAME}_gencfg)
target_link_libraries(jsk_pcl_ros_utils
  ${catkin_LIBRARIES} ${pcl_ros_LIBRARIES} ${OpenCV_LIBRARIES} yaml-cpp)

# bench/ directory
# generate_messages(DEPENDENCIES
#   ${PCL_MSGS} sensor_msgs geometry_msgs jsk_recognition_msgs jsk_footstep_msgs)

get_property(dirs DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)
message("flags: ${CMAKE_CXX_FLAGS}")

catkin_package(
    DEPENDS pcl
    CATKIN_DEPENDS pcl_ros message_runtime ${PCL_MSGS} sensor_msgs geometry_msgs jsk_recognition_msgs jsk_recognition_utils
    INCLUDE_DIRS include
    LIBRARIES jsk_pcl_ros_utils
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
install(TARGETS jsk_pcl_ros_utils
  ${jsk_pcl_util_nodelet_executables}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

install(FILES jsk_pcl_nodelets.xml DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})