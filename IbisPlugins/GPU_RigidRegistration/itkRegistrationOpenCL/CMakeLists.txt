#================================
# Define sources
#================================
SET( IBIS_ITK_REGISTRATION_OPENCL_SRC
    itkGPUOrientationMatchingMatrixTransformationSparseMask.hxx
    itkGPU3DRigidSimilarityMetric.h
)

SET( IBIS_ITK_REGISTRATION_OPENCL_HDR
    itkGPUOrientationMatchingMatrixTransformationSparseMask.h
)

set( LibCL GPUDiscreteGaussianGradientImageFilter 
    GPUOrientationMatchingMatrixTransformationSparseMaskKernel
)

#================================
# Create custom commands to
# encode each cl file into a
# C string literal in a header
# file.
#================================
foreach( file ${LibCL} )
  set( clKernel_src ${CMAKE_CURRENT_SOURCE_DIR}/${file}.cl )
  set( clKernel_cxx ${CMAKE_CURRENT_BINARY_DIR}/${file}.cxx )
  set( clKernel_h ${CMAKE_CURRENT_BINARY_DIR}/${file}.h )
  add_custom_command(
    OUTPUT ${clKernel_cxx} ${clKernel_h}
    DEPENDS ${clKernel_src}
    COMMAND ${VTK_ENCODESTRING_EXE}
    ARGS ${clKernel_cxx} ${clKernel_src} ${file} --build-header " " vtkSystemIncludes.h )
  set( LibCLSrc ${LibCLSrc} ${clKernel_src} ${clKernel_h} ${clKernel_cxx} )
  set( LibCLHdr ${LibCLHdr} ${clKernel_h} )
endforeach(file)

#================================
# Define output
#================================
IF (BUILD_SHARED_LIBS)
   ADD_LIBRARY( itkRegistrationOpenCL SHARED ${IBIS_ITK_REGISTRATION_OPENCL_SRC} ${IBIS_ITK_REGISTRATION_OPENCL_HDR} ${LibCLSrc} )
ELSE( BUILD_SHARED_LIBS )
   ADD_LIBRARY( itkRegistrationOpenCL ${IBIS_ITK_REGISTRATION_OPENCL_SRC} ${IBIS_ITK_REGISTRATION_OPENCL_HDR} ${LibCLSrc} )
ENDIF( BUILD_SHARED_LIBS )

target_link_libraries( itkRegistrationOpenCL ${ITK_LIBRARIES}  ${OPENCL_LIBRARIES})

#================================
# Define include dir for
# dependent projects
#================================
target_include_directories( itkRegistrationOpenCL PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR} ${OPENCL_INCLUDE_DIRS} )
