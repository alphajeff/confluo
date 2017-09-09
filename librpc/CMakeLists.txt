set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-parameter")

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
file(MAKE_DIRECTORY ${EXECUTABLE_OUTPUT_PATH})

set(RPC_INCLUDE rpc)
set(UTILS_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/../libutils/utils)
set(DIALOG_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/../libdialog/dialog)

include_directories(${gtest_SOURCE_DIR}/include
  ${RPC_INCLUDE}
  ${UTILS_INCLUDE}
  ${DIALOG_INCLUDE}
  ${THRIFT_INCLUDE}
  ${THRIFT_BUILD_INCLUDE}
  ${Boost_INCLUDE_DIR})
  
# Build server executable
set(SERVER_SOURCES src/dialog_constants.cc
                   src/dialog_service.cc 
                   src/dialog_server.cc 
                   src/dialog_types.cc)
add_executable(dialogd ${SERVER_SOURCES})
target_link_libraries(dialogd ${CMAKE_THREAD_LIBS_INIT} thriftstatic)
add_dependencies(dialogd thrift)

set(CLIENT_SOURCES src/dialog_constants.cc 
                   src/dialog_service.cc 
                   src/dialog_types.cc)
add_library(dclient STATIC ${CLIENT_SOURCES})
target_link_libraries(dclient thriftstatic)
add_dependencies(dclient thrift)

if (BUILD_TESTS)
  # Build test
  file(GLOB_RECURSE TEST_SOURCES test/*.cc)
  add_executable(rpctest ${TEST_SOURCES})
  target_link_libraries(rpctest ${TEST_LINK_LIBS} dclient thriftstatic)
  add_dependencies(rpctest googletest thrift)
  
  # Register test
  enable_testing()
  add_test(RpcTest ${EXECUTABLE_OUTPUT_PATH}/rpctest)
endif()

# install
install(DIRECTORY rpc/
  DESTINATION include
  FILES_MATCHING PATTERN "*")

install(TARGETS dialogd dclient
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)