PROJECT(CHOLMOD)

include_directories("${suitesparseconfig_SOURCE_DIR}")
include_directories("${AMD_SOURCE_DIR}/Include")
include_directories("${CAMD_SOURCE_DIR}/Include")
include_directories("${COLAMD_SOURCE_DIR}/Include")
include_directories("${CCOLAMD_SOURCE_DIR}/Include")

# Headers:
include_directories("${CHOLMOD_SOURCE_DIR}/Include")
FILE(GLOB LIBHDRS "${CHOLMOD_SOURCE_DIR}/Include/*.h")

FILE(GLOB LIBSRCS "SourceWrappers/*.c")

declare_suitesparse_library(cholmod "${LIBSRCS}" "${LIBHDRS}"
	TARGET_PRIVATE_LINK	${SuiteSparse_LINKER_LAPACK_BLAS_LIBS}
	TARGET_PUBLIC_LINK 	${SuiteSparse_LINKER_METIS_LIBS} amd camd colamd ccolamd
)

# CUDA modifications
if(SuiteSparse_GPU)
   find_package(CUDA)
   set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-O3 -gencode=arch=compute_20,code=sm_20 -gencode=arch=compute_30,code=sm_30 -gencode=arch=compute_35,code=sm_35)
   cuda_include_directories("${suitesparseconfig_SOURCE_DIR}")
   cuda_compile(CHOLMOD_GPU_CU_FILES "${CHOLMOD_SOURCE_DIR}/GPU/cholmod_gpu_kernels.cu")
   file(GLOB CHOLMOD_GPU_C_FILES "${CHOLMOD_SOURCE_DIR}/GPU/*.c")

   cuda_add_library(cholmod_cuda ${CHOLMOD_GPU_CU_FILES} ${CHOLMOD_GPU_C_FILES})
   cuda_add_cublas_to_target(cholmod_cuda)
endif(SuiteSparse_GPU)
# End CUDA modifications
