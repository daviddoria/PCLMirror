set(SUBSYS_NAME modeler)
set(SUBSYS_DESC "PCLModeler: PCL based reconstruction platform")
set(SUBSYS_DEPS common geometry io filters sample_consensus segmentation visualization kdtree features surface octree registration keypoints tracking search recognition)
set(DEFAULT OFF)
set(REASON "")

# Find VTK and QVTK
if(VTK_FOUND AND VTK_USE_QVTK)
  set(DEFAULT TRUE)
  set(REASON)
  set(VTK_USE_FILE ${VTK_USE_FILE} CACHE INTERNAL "VTK_USE_FILE")
  include (${VTK_USE_FILE})
elseif(NOT VTK_FOUND)
  set(DEFAULT FALSE)
  set(REASON "VTK was not found.")
elseif(NOT VTK_USE_QVTK)
  set(DEFAULT FALSE)
  set(REASON "VTK was not built with Qt support.")
endif(VTK_FOUND AND VTK_USE_QVTK)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

PCL_SUBSYS_OPTION(build ${SUBSYS_NAME} ${SUBSYS_DESC} ${DEFAULT} ${REASON})
PCL_SUBSYS_DEPEND(build ${SUBSYS_NAME} DEPS ${SUBSYS_DEPS} EXT_DEPS vtk)

PCL_ADD_DOC(${SUBSYS_NAME})

if(build)

  include_directories (${CMAKE_CURRENT_BINARY_DIR})
  include_directories (${CMAKE_CURRENT_SOURCE_DIR}/include)

  # Set Qt files and resources here  
  set(uis main_window.ui)
  set(moc_incs include/pcl/apps/${SUBSYS_NAME}/main_window.h)
  set(resources resources/resources.qrc)

  set(incs ${moc_incs}
           include/pcl/apps/${SUBSYS_NAME}/cloud_actor.h
           include/pcl/apps/${SUBSYS_NAME}/pcl_modeler.h
           include/pcl/apps/${SUBSYS_NAME}/render_widget.h
           include/pcl/apps/${SUBSYS_NAME}/dock_widget.h)

  set(srcs src/main.cpp
           src/main_window.cpp
           src/cloud_actor.cpp
           src/pcl_modeler.cpp
           src/render_widget.cpp
           src/dock_widget.cpp)

  set(impl_incs include/pcl/apps/${SUBSYS_NAME}/impl/pcl_modeler.hpp)

  # Qt stuff
  QT4_WRAP_UI(ui_srcs ${uis})
  QT4_WRAP_CPP(moc_srcs ${moc_incs})
  QT4_ADD_RESOURCES(resource_srcs ${resources})

  # Organize files
  SOURCE_GROUP("Resources" FILES ${uis} ${resources} ${EXE_ICON})
  SOURCE_GROUP("Generated" FILES ${ui_srcs} ${moc_srcs} ${resource_srcs} ${RCS_SOURCES})
  SET_SOURCE_FILES_PROPERTIES(${srcs} PROPERTIES OBJECT_DEPENDS "${ui_srcs}")

  # Generate executable
  set(EXE_NAME pcl_${SUBSYS_NAME})
  PCL_ADD_EXECUTABLE(${EXE_NAME} ${SUBSYS_NAME} ${ui_srcs} ${moc_srcs} ${resource_srcs} ${srcs} ${incs} ${impl_incs})
  target_link_libraries(${EXE_NAME} pcl_common pcl_io pcl_kdtree pcl_filters pcl_visualization pcl_segmentation pcl_surface pcl_features pcl_sample_consensus pcl_search QVTK ${QT_LIBRARIES})

  # Put the ui in the windows project file
  IF (${CMAKE_BUILD_TOOL} MATCHES "msdev")
    SET (srcs ${srcs} ${uis})
  ENDIF (${CMAKE_BUILD_TOOL} MATCHES "msdev")
  IF (${CMAKE_BUILD_TOOL} MATCHES "devenv")
    SET (srcs ${srcs} ${uis})
  ENDIF (${CMAKE_BUILD_TOOL} MATCHES "devenv")

  # Install include files
  PCL_ADD_INCLUDES(${SUBSYS_NAME} ${SUBSYS_NAME} ${incs})
  PCL_ADD_INCLUDES(${SUBSYS_NAME} ${SUBSYS_NAME}/impl ${impl_incs})

  PCL_MAKE_PKGCONFIG(${EXE_NAME} ${SUBSYS_NAME} "${SUBSYS_DESC}" "" "" "" "" "")

  add_subdirectory(tools)

endif(build)
