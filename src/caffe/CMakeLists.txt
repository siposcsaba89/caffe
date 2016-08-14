# generate protobuf sources
file(GLOB proto_files proto/*.proto)
caffe_protobuf_generate_cpp_py(${proto_gen_folder} proto_srcs proto_hdrs proto_python ${proto_files})

# include python files either to force generation
add_library(proto STATIC ${proto_hdrs} ${proto_srcs} ${proto_python})
target_compile_definitions(proto PUBLIC -DBOOST_ALL_NO_LIB=1)
target_link_libraries(proto protobuf::libprotobuf)
#target_include_directories(proto PRIVATE ${PROTOBUF_INCLUDE_DIR})
set(Caffe_LINKER_LIBS proto ${Caffe_LINKER_LIBS}) # note, crucial to prepend!
caffe_default_properties(proto)

# --[ Caffe library

# creates 'test_srcs', 'srcs', 'test_cuda', 'cuda' lists
caffe_pickup_caffe_sources(${PROJECT_SOURCE_DIR})

if(USE_CUDA AND HAVE_CUDA)
  caffe_cuda_compile(cuda_objs ${cuda})
  list(APPEND srcs ${cuda_objs} ${cuda})
 else()
# list(REMOVE_ITEM ${srcs} *.cu)
endif()

add_library(caffe ${srcs})
#message(FATAL_ERROR ${srcs})
target_link_libraries(caffe PUBLIC proto ${Caffe_LINKER_LIBS} hdf5_hl-static hdf5-static protobuf::libprotobuf glog::glog)
caffe_default_properties(caffe)
set_target_properties(caffe PROPERTIES
    VERSION   ${CAFFE_TARGET_VERSION}
    SOVERSION ${CAFFE_TARGET_SOVERSION}
    )
target_compile_definitions(caffe PUBLIC -DBOOST_ALL_NO_LIB=1)
# ---[ Tests
 add_subdirectory(test)

# ---[ Install
install(DIRECTORY ${Caffe_INCLUDE_DIR}/caffe DESTINATION include)
install(FILES ${proto_hdrs} DESTINATION include/caffe/proto)
install(TARGETS caffe proto EXPORT CaffeTargets DESTINATION lib)

file(WRITE ${PROJECT_BINARY_DIR}/__init__.py)
list(APPEND proto_python ${PROJECT_BINARY_DIR}/__init__.py)
install(PROGRAMS ${proto_python} DESTINATION python/caffe/proto)


