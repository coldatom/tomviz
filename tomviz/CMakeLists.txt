# We need to link to QtMain on Windows, this is much better in Qt 5 with recent
# CMake versions where a policy can invoke auto-linking of this library.
if(WIN32)
  find_package(Qt4 4.8 COMPONENTS QtGui QtMain REQUIRED)
endif()

option(ENABLE_DAX_ACCELERATION "Enable Accelerated Algorithms" OFF)
if(ENABLE_DAX_ACCELERATION)
  find_package(Dax REQUIRED)
  DaxConfigureTBB(REQUIRED)

  add_subdirectory(dax)

  include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dax)
  set(accel_srcs
    dax/ModuleStreamingContour.cxx
    dax/ModuleAccelThreshold.cxx
    )
endif()

add_subdirectory(pvextensions)

set(SOURCES
  ActiveObjects.cxx
  ActiveObjects.h
  AddAlignReaction.cxx
  AddAlignReaction.h
  AddExpressionReaction.cxx
  AddExpressionReaction.h
  AddPythonTransformReaction.cxx
  AddRenderViewContextMenuBehavior.cxx
  AddRenderViewContextMenuBehavior.h
  AddResampleReaction.cxx
  AddResampleReaction.h
  AddRotateAlignReaction.cxx
  AddRotateAlignReaction.h
  AlignWidget.cxx
  AlignWidget.h
  Behaviors.cxx
  Behaviors.h
  CentralWidget.cxx
  CentralWidget.h
  CloneDataReaction.cxx
  CloneDataReaction.h
  CropReaction.cxx
  CropReaction.h
  CropOperator.cxx
  CropOperator.h
  SelectVolumeWidget.cxx
  SelectVolumeWidget.h
  DataPropertiesPanel.cxx
  DataPropertiesPanel.h
  DataSource.cxx
  DataSource.h
  DeleteDataReaction.cxx
  DeleteDataReaction.h
  EditOperatorDialog.cxx
  EditOperatorDialog.h
  EditOperatorWidget.cxx
  EditOperatorWidget.h
  LoadDataReaction.cxx
  LoadDataReaction.h
  main.cxx
  MainWindow.cxx
  MainWindow.h
  Module.cxx
  Module.h
  ModuleContour.cxx
  ModuleContour.h
  ModuleFactory.cxx
  ModuleFactory.h
  ModuleManager.cxx
  ModuleManager.h
  ModuleMenu.cxx
  ModuleMenu.h
  ModuleOrthogonalSlice.cxx
  ModuleOrthogonalSlice.h
  ModuleOutline.cxx
  ModuleOutline.h
  ModulePropertiesPanel.cxx
  ModulePropertiesPanel.h
  ModuleSegment.cxx
  ModuleSegment.h
  ModuleSlice.cxx
  ModuleSlice.h
  ModuleThreshold.cxx
  ModuleThreshold.h
  ModuleVolume.cxx
  ModuleVolume.h
  Operator.cxx
  Operator.h
  OperatorPython.cxx
  OperatorPython.h
  OperatorsWidget.cxx
  OperatorsWidget.h
  PipelineWidget.cxx
  PipelineWidget.h
  ProgressBehavior.cxx
  ProgressBehavior.h
  PythonGeneratedDatasetReaction.cxx
  PythonGeneratedDatasetReaction.h
  RecentFilesMenu.cxx
  RecentFilesMenu.h
  ReconstructionReaction.h
  ReconstructionReaction.cxx
  ResetReaction.cxx
  ResetReaction.h
  RotateAlignWidget.cxx
  RotateAlignWidget.h
  SaveDataReaction.cxx
  SaveLoadStateReaction.cxx
  SaveLoadStateReaction.h
  ScaleActorBehavior.cxx
  ScaleActorBehavior.h
  SetScaleReaction.cxx
  SetScaleReaction.h
  SetTiltAnglesReaction.cxx
  SetTiltAnglesReaction.h
  ToggleDataTypeReaction.h
  ToggleDataTypeReaction.cxx
  TomographyReconstruction.h
  TomographyReconstruction.cxx
  TomographyTiltSeries.h
  TomographyTiltSeries.cxx
  Utilities.cxx
  Utilities.h
  ViewPropertiesPanel.cxx
  ViewPropertiesPanel.h
  ViewMenuManager.cxx
  ViewMenuManager.h
  vtkNonOrthoImagePlaneWidget.cxx
  vtkNonOrthoImagePlaneWidget.h
  )

set(python_files
  MisalignImgs_Uniform.py
  Align_Images.py
  Recon_DFT.py
  Recon_WBP.py
  Recon_ART.py
  Crop_Data.py
  DownsampleByTwo.py
  FFT_AbsLog.py
  Shift_Stack_Uniformly.py
  Shift3D.py
  Square_Root_Data.py
  Subtract_TiltSer_Background.py
  MisalignImgs_Gaussian.py
  Rotate3D.py
  HannWindow3D.py
  SobelFilter.py
  LaplaceFilter.py
  GaussianFilter.py
  MedianFilter.py
  Resample.py
  deleteSlices.py
  ClearVolume.py
  ConstantDataset.py
  RandomParticles.py
  TiltSeries.py 
  )

# Make symlinks where possible, file copies where necessary in the build tree.
if(UNIX)
  set(script_cmd "create_symlink")
else()
  set(script_cmd "copy_if_different")
endif()
make_directory("${tomviz_BINARY_DIR}/share/tomviz/scripts")
foreach(file ${python_files})
  list(APPEND SOURCES "python/${file}")
  execute_process(COMMAND ${CMAKE_COMMAND} -E ${script_cmd}
    "${tomviz_SOURCE_DIR}/tomviz/python/${file}"
    "${tomviz_BINARY_DIR}/share/tomviz/scripts/${file}")
  install(FILES "python/${file}"
    DESTINATION "${tomviz_data_install_dir}/scripts"
    COMPONENT "Scripts")
endforeach()

qt4_wrap_ui(UI_SOURCES
  AboutDialog.ui
  CentralWidget.ui
  SelectVolumeWidget.ui
  DataPropertiesPanel.ui
  EditPythonOperatorWidget.ui
  MainWindow.ui
  ModulePropertiesPanel.ui
  RotateAlignWidget.ui
  ViewPropertiesPanel.ui
  )
qt4_add_resources(RCC_SOURCES resources.qrc)

if(APPLE)
  list(APPEND SOURCES icons/tomviz.icns)
  set(MACOSX_BUNDLE_ICON_FILE tomviz.icns)
  set(MACOSX_BUNDLE_BUNDLE_VERSION "${tomviz_version}")
  set_source_files_properties(icons/tomviz.icns PROPERTIES
    MACOSX_PACKAGE_LOCATION Resources)
elseif(WIN32)
  list(APPEND SOURCES icons/tomviz.rc)
endif()

include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})

configure_file(tomvizConfig.h.in tomvizConfig.h @ONLY)
configure_file(tomvizPythonConfig.h.in tomvizPythonConfig.h @ONLY)

add_executable(tomviz WIN32 MACOSX_BUNDLE ${SOURCES} ${UI_SOURCES}
  ${RCC_SOURCES} ${accel_srcs})

set_target_properties(tomviz PROPERTIES AUTOMOC TRUE)

target_link_libraries(tomviz
  LINK_PRIVATE
    pqApplicationComponents
    vtkPVServerManagerRendering
    vtkpugixml
    tomvizExtensions
  )
if(WIN32)
  target_link_libraries(tomviz LINK_PRIVATE ${QT_QTMAIN_LIBRARY})
endif()
if(APPLE)
  install(TARGETS tomviz DESTINATION Applications COMPONENT runtime)
else()
  install(TARGETS tomviz DESTINATION bin COMPONENT runtime)
endif()

# Install the tomviz Python files.
install(DIRECTORY python/tomviz
       DESTINATION "${tomviz_python_install_dir}"
       USE_SOURCE_PERMISSIONS
       COMPONENT runtime)
if(tomviz_data_DIR)
  add_definitions(-DTOMVIZ_DATA)
  install(DIRECTORY "${tomviz_data_DIR}"
    DESTINATION "${tomviz_data_install_dir}"
    USE_SOURCE_PERMISSIONS
    COMPONENT runtime)
endif()

if(ENABLE_DAX_ACCELERATION)
  target_link_libraries(tomviz
    LINK_PRIVATE
      tomvizStreaming
      tomvizThreshold
      ${TBB_LIBRARIES})

  #set the dax backend to tbb explicitly as the histogram is
  #computed using dax.
  set_target_properties(tomviz PROPERTIES COMPILE_DEFINITIONS
    "DAX_DEVICE_ADAPTER=DAX_DEVICE_ADAPTER_TBB")
endif()
