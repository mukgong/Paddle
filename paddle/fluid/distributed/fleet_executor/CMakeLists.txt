proto_library(fleet_executor_desc_proto SRCS fleet_executor_desc.proto)
if(WITH_PYTHON)
  py_proto_compile(fleet_executor_desc_py_proto SRCS fleet_executor_desc.proto)
endif()
proto_library(interceptor_message_proto SRCS interceptor_message.proto)

if(WITH_ARM_BRPC)
  set(BRPC_DEPS arm_brpc snappy gflags glog)
elseif(WITH_DISTRIBUTE AND WITH_PSCORE)
  set(BRPC_DEPS brpc ssl crypto protobuf zlib leveldb snappy gflags glog)
else()
  set(BRPC_DEPS "")
endif()

cc_library(task_loop_thread_pool SRCS task_loop_thread_pool.cc task_loop_thread.cc task_loop.cc DEPS enforce glog)

cc_library(fleet_executor SRCS fleet_executor.cc carrier.cc task_node.cc runtime_graph.cc dist_model.cc interceptor.cc
        compute_interceptor.cc amplifier_interceptor.cc source_interceptor.cc sink_interceptor.cc message_service.cc message_bus.cc dist_model_tensor_wrapper.cc
        DEPS proto_desc fleet_executor_desc_proto interceptor_message_proto task_loop_thread_pool collective_helper
        op_registry executor_gc_helper gflags glog ${BRPC_DEPS})

if(WITH_DISTRIBUTE)
  set(DISTRIBUTE_COMPILE_FLAGS "-Wno-non-virtual-dtor -Wno-error=non-virtual-dtor -Wno-error=delete-non-virtual-dtor")
  if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 7.0)
    set(DISTRIBUTE_COMPILE_FLAGS "${DISTRIBUTE_COMPILE_FLAGS} -faligned-new")
  endif()
  set_source_files_properties(interceptor.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(compute_interceptor.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(amplifier_interceptor.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(source_interceptor.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(sink_interceptor.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(message_bus.h PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(message_bus.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(fleet_executor.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(carrier.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(message_service.h PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})
  set_source_files_properties(message_service.cc PROPERTIES COMPILE_FLAGS ${DISTRIBUTE_COMPILE_FLAGS})

  add_subdirectory(test)
endif()
