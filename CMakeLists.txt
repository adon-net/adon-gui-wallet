cmake_minimum_required(VERSION 2.8.6)
include(CryptoNoteWallet.cmake)
include(QREncode.cmake)

cmake_policy(SET CMP0043 NEW)
cmake_policy(SET CMP0048 NEW)
#cmake_policy(SET CMP0071 NEW)
#cmake_policy(SET CMP0028 NEW)
cmake_policy(SET CMP0020 NEW)
enable_language(C)
enable_language(CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project(${CN_PROJECT_NAME})
## This section is for settings found in the slow-hash routine(s) that may benefit some systems (mostly ARM)
set(FORCE_USE_HEAP ON CACHE BOOL "Force the use of heap memory allocation")
set(NO_AES OFF CACHE BOOL "Turn off Hardware AES instructions?")
set(NO_OPTIMIZED_MULTIPLY_ON_ARM OFF CACHE BOOL "Turn off Optimized Multiplication on ARM?")

message(STATUS "Building for target architecture: ${ARCH}")

if (FORCE_USE_HEAP)
    add_definitions(-DFORCE_USE_HEAP)
    message(STATUS "FORCE_USE_HEAP: ENABLED")
else ()
    message(STATUS "FORCE_USE_HEAP: DISABLED")
endif ()

if (NO_AES)
    add_definitions(-DNO_AES)
    message(STATUS "HW AES: DISABLED")
else ()
    message(STATUS "HW AES: ENABLED")
endif ()

if (NO_OPTIMIZED_MULTIPLY_ON_ARM)
    add_definitions(-DNO_OPTIMIZED_MULTIPLY_ON_ARM)
    message(STATUS "OPTIMIZED_ARM_MULTIPLICATION: DISABLED")
else ()
    message(STATUS "OPTIMIZED_ARM_MULTIPLICATION: ENABLED")
endif ()
execute_process(COMMAND git log -1 --pretty=format:%h
WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
OUTPUT_VARIABLE GIT_REVISION)

execute_process(COMMAND git log -1 --pretty=format:%h
WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/cryptonote/"
OUTPUT_VARIABLE CN_REVISION)

set(VERSION "${CN_REVISION}")
configure_file("cryptonote/src/version.h.in" "version.h")
configure_file("src/CryptoNoteWalletConfig.h.in" "CryptoNoteWalletConfig.h")

add_definitions (-DGIT_REVISION="${GIT_REVISION}")
add_definitions (-DCN_VERSION="${CN_VERSION}")
add_definitions (-DCN_REVISION="${CN_REVISION}")

set(CRYPTONOTE_LIB cryptonote)

include_directories(${CMAKE_BINARY_DIR}
  src
  include
  cryptonote/external
  cryptonote/include
  cryptonote/src
  libqrencode
)

find_package(Qt5Gui REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5Network REQUIRED)
find_package(Qt5DBus REQUIRED)

# include_directories(${Qt5DBus_INCLUDE_DIRS} cryptonote/external/rocksdb/include/ include src external "${CMAKE_CURRENT_BINARY_DIR}/version")

set(Boost_USE_STATIC_LIBS ON)
if(WIN32)
  set(Boost_USE_STATIC_RUNTIME OFF)
else(WIN32)
  set(Boost_USE_STATIC_RUNTIME ON)
endif(WIN32)

find_package(Boost 1.58 REQUIRED COMPONENTS date_time filesystem program_options regex serialization system thread chrono)
if ((${Boost_MAJOR_VERSION} EQUAL 1) AND (${Boost_MINOR_VERSION} EQUAL 54))
  message(SEND_ERROR "Boost version 1.58 is unsupported, more details are available here http://goo.gl/RrCFmA")
endif ()

include_directories(${Boost_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

add_definitions(-DGIT_REVISION="${GIT_REVISION}")

set(CMAKE_AUTOMOC ON)

file(GLOB_RECURSE CRYPTONOTE_SOURCES
  cryptonote/external/miniupnpc/connecthostport.c
  cryptonote/external/miniupnpc/igd_desc_parse.c
  cryptonote/external/miniupnpc/minisoap.c
  cryptonote/external/miniupnpc/miniupnpc.c
  cryptonote/external/miniupnpc/miniwget.c
  cryptonote/external/miniupnpc/minixml.c
  cryptonote/external/miniupnpc/portlistingparse.c
  cryptonote/external/miniupnpc/receivedata.c
  cryptonote/external/miniupnpc/upnpcommands.c
  cryptonote/external/miniupnpc/upnpreplyparse.c
  cryptonote/src/BlockchainExplorer/BlockchainExplorer.cpp
  cryptonote/src/BlockchainExplorer/BlockchainExplorerErrors.cpp
  cryptonote/src/BlockchainExplorer/BlockchainExplorerDataBuilder.cpp
  cryptonote/src/Common/Base58.cpp
  cryptonote/src/Common/CommandLine.cpp
  cryptonote/src/Common/Util.cpp
  cryptonote/src/Common/StringTools.cpp
  cryptonote/src/Common/JsonValue.cpp
  cryptonote/src/Common/ConsoleTools.cpp
  cryptonote/src/Common/MemoryInputStream.cpp
  cryptonote/src/Common/PathTools.cpp
  cryptonote/src/Common/DnsTools.cpp
  cryptonote/src/Common/StdInputStream.cpp
  cryptonote/src/Common/StdOutputStream.cpp
  cryptonote/src/Common/StreamTools.cpp
  cryptonote/src/Common/StringOutputStream.cpp
  cryptonote/src/Common/StringView.cpp
  cryptonote/src/Common/VectorOutputStream.cpp
  cryptonote/src/Common/ScopeExit.cpp
  cryptonote/src/ConnectivityTool/ConnectivityTool.cpp
  cryptonote/src/crypto/aesb.c
  cryptonote/src/crypto/crypto-ops-data.c
  cryptonote/src/crypto/groestl.c
  cryptonote/src/crypto/hash-extra-jh.c
  cryptonote/src/crypto/initializer.h
  cryptonote/src/crypto/oaes_config.h
  cryptonote/src/crypto/skein.h
  cryptonote/src/crypto/blake256.c
  cryptonote/src/crypto/crypto-ops.c
  cryptonote/src/crypto/groestl.h
  cryptonote/src/crypto/hash-extra-skein.c
  cryptonote/src/crypto/jh.c
  cryptonote/src/crypto/oaes_lib.c
  cryptonote/src/crypto/skein_port.h
  cryptonote/src/crypto/blake256.h
  cryptonote/src/crypto/crypto-ops.h
  cryptonote/src/crypto/groestl_tables.h
  cryptonote/src/crypto/hash-ops.h
  cryptonote/src/crypto/jh.h
  cryptonote/src/crypto/oaes_lib.h
  cryptonote/src/crypto/slow-hash.c
  cryptonote/src/crypto/chacha8.cpp
  cryptonote/src/crypto/crypto.cpp
  cryptonote/src/crypto/hash-extra-blake.c
  cryptonote/src/crypto/hash.c
  cryptonote/src/crypto/keccak.c
  cryptonote/src/crypto/random.h
  cryptonote/src/crypto/tree-hash.c
  cryptonote/src/crypto/chacha8.h
  cryptonote/src/crypto/crypto.h
  cryptonote/src/crypto/hash-extra-groestl.c hash.h
  cryptonote/src/crypto/keccak.h
  cryptonote/src/crypto/skein.c
  cryptonote/src/crypto/variant2_int_sqrt.h
  cryptonote/src/CryptoNoteCore/*.cpp
  cryptonote/src/CryptoNoteProtocol/*.cpp
  cryptonote/src/Daemon/*.cpp
  cryptonote/src/HTTP/*.cpp
  cryptonote/src/InProcessNode/InProcessNode.cpp
  cryptonote/src/InProcessNode/InProcessNodeErrors.cpp
  cryptonote/src/Logging/*.cpp
  cryptonote/src/NodeRpcProxy/NodeErrors.cpp
  cryptonote/src/NodeRpcProxy/NodeRpcProxy.cpp
  cryptonote/src/P2p/*.cpp
  cryptonote/src/Rpc/*.cpp
  cryptonote/src/Serialization/*.cpp
  cryptonote/src/Transfers/BlockchainSynchronizer.cpp
  cryptonote/src/Transfers/SynchronizationState.cpp
  cryptonote/src/Transfers/TransfersConsumer.cpp
  cryptonote/src/Transfers/TransfersContainer.cpp
  cryptonote/src/Transfers/TransfersSubscription.cpp
  cryptonote/src/Transfers/TransfersSynchronizer.cpp
  cryptonote/src/Wallet/*.cpp
  cryptonote/src/WalletLegacy/KeysStorage.cpp
  cryptonote/src/WalletLegacy/WalletLegacy.cpp
  cryptonote/src/WalletLegacy/WalletHelper.cpp
  cryptonote/src/WalletLegacy/WalletLegacySerializer.cpp
  cryptonote/src/WalletLegacy/WalletLegacySerialization.cpp
  cryptonote/src/WalletLegacy/WalletTransactionSender.cpp
  cryptonote/src/WalletLegacy/WalletUnconfirmedTransactions.cpp
  cryptonote/src/WalletLegacy/WalletUserTransactionsCache.cpp
  cryptonote/src/System/ContextGroup.cpp
  cryptonote/src/System/Event.cpp
  cryptonote/src/System/EventLock.cpp
  cryptonote/src/System/InterruptedException.cpp
  cryptonote/src/System/Ipv4Address.cpp
  cryptonote/src/System/TcpStream.cpp
  cryptonote/src/JsonRpcServer/*.cpp
  cryptonote/src/PaymentGate/*.cpp
  cryptonote/src/PaymentGateService/*.cpp
  cryptonote/src/Miner/*.cpp
)

file(GLOB_RECURSE Mnemonics cryptonote/src/Mnemonics/*)

file(GLOB_RECURSE SOURCES src/*.cpp)
file(GLOB_RECURSE HEADERS src/*.h)
file(GLOB_RECURSE FORMS src/gui/ui/*.ui)

if(MSVC)
  include_directories(Platform/Windows)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Windows/ApplicationEventHandler.cpp)
elseif(APPLE)
  include_directories(Platform/OSX)
  include_directories(Platform/Posix)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/OSX/ApplicationEventHandler.h ${CMAKE_SOURCE_DIR}/Platform/OSX/ApplicationEventHandler.mm ${CMAKE_SOURCE_DIR}/Platform/OSX/INotificationMonitorObserver.h ${CMAKE_SOURCE_DIR}/Platform/OSX/NotificationMonitor.h ${CMAKE_SOURCE_DIR}/Platform/OSX/NotificationMonitor.mm ${CMAKE_SOURCE_DIR}/Platform/OSX/WalletApplication.mm )
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Posix/System/MemoryMappedFile.cpp)
else()
  include_directories(Platform/Linux)
  include_directories(Platform/Posix)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Linux/ApplicationEventHandler.cpp)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Posix/System/MemoryMappedFile.cpp)
endif()


set(QRC src/resources.qrc)

qt5_wrap_ui(UIS ${FORMS})
qt5_add_resources(RCC ${QRC})

if (WIN32)
  if (NOT MSVC)
    message(FATAL_ERROR "Only MSVC is supported on this platform")
  endif ()

  set(STATIC ${MSVC} CACHE BOOL "Link libraries statically")

  add_definitions(/D_CRT_SECURE_NO_WARNINGS /D_WIN32_WINNT=0x0600 /DSTATICLIB)
  include_directories(cryptonote/src/Platform/msc)

  set( CMAKE_CXX_FLAGS "-bigobj -EHsc")
  set(PLATFORM_DIR Windows)
  set(BUILD_PLATFORM WIN32)
  set(BUILD_RESOURCES src/adon.rc)
  set(QTMAIN Qt5::WinMain)

elseif (UNIX)

  set(CRYPTONOTE_SOURCES ${CRYPTONOTE_SOURCES} cryptonote/external/miniupnpc/minissdpc.c)

  if (APPLE)
    enable_language(ASM)
    file(GLOB_RECURSE OBJC_SOURCES src/*.mm)
    set(SOURCES ${SOURCES} ${OBJC_SOURCES})
    set(PLATFORM_DIR OSX)
    set(MACOSX_BUNDLE_INFO_STRING "Adon GUI wallet")
    set(MACOSX_BUNDLE_LONG_VERSION_STRING "${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")
    set(MACOSX_BUNDLE_BUNDLE_NAME Adon)
    set(MACOSX_BUNDLE_SHORT_VERSION_STRING "${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")
    set(MACOSX_BUNDLE_BUNDLE_VERSION "$${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")

    find_package(Qt5PrintSupport REQUIRED)

    include_directories(/usr/include/malloc)

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maes -std=c++11 -stdlib=libc++")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11 -maes -D_DARWIN_C_SOURCE")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -framework Cocoa -framework OpenGL -framework CoreFoundation -framework Carbon -framework IOKit -L/usr/lib")

    set(MACOSX_BUNDLE_ICON_FILE adon.icns)
    set(APPLICATION_ICON src/images/adon.icns)
    set_source_files_properties(${APPLICATION_ICON} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")

    set(BUILD_PLATFORM MACOSX_BUNDLE)
    set(BUILD_RESOURCES ${APPLICATION_ICON})

    GET_TARGET_PROPERTY(QT_LIB_DIR "${Qt5Widgets_LIBRARIES}" LOCATION)
    GET_FILENAME_COMPONENT(QT_LIB_DIR "${QT_LIB_DIR}" PATH)

  else(APPLE)

    set(PLATFORM_DIR Linux)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maes -std=c++11")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -maes -std=c11")
  endif (APPLE)
endif ()

include_directories(cryptonote/src/Platform/${PLATFORM_DIR})
file(GLOB PLATFORM_SOURCES cryptonote/src/Platform/${PLATFORM_DIR}/System/*)
set(CRYPTONOTE_SOURCES ${CRYPTONOTE_SOURCES} ${PLATFORM_SOURCES} ${CRYPTONOTE_PLATFORM})

add_library(${CRYPTONOTE_LIB} STATIC ${CRYPTONOTE_SOURCES})
add_library(Mnemonics ${Mnemonics})
set_target_properties(${CRYPTONOTE_LIB} PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)

if (APPLE OR ${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
  target_link_libraries(${CRYPTONOTE_LIB} ${Boost_LIBRARIES} ${Qt5DBus_LIBRARIES} -lresolv)
else ()
  target_link_libraries(${CRYPTONOTE_LIB} ${Boost_LIBRARIES} ${Qt5DBus_LIBRARIES})
endif ()

set_target_properties(Mnemonics PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)
target_link_libraries(Mnemonics ${Boost_LIBRARIES})

add_executable(${PROJECT_NAME} ${BUILD_PLATFORM} ${BUILD_RESOURCES} ${SOURCES} ${HEADERS} ${UIS} ${RCC})

set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES} ${QTMAIN} ${CRYPTONOTE_LIB} ${Qt5DBus_LIBRARIES} ${QRENCODE_LIB}  ${QRENCODE_LIB} Mnemonics)

if (APPLE)
  qt5_use_modules(${PROJECT_NAME} PrintSupport)
elseif (UNIX)
  target_link_libraries(${PROJECT_NAME} -lpthread)
elseif (WIN32)
  target_link_libraries(${PROJECT_NAME} Imm32 Iphlpapi Winmm UxTheme d3d9 dxguid Rpcrt4 Wtsapi32 WSock32 ws2_32 Dwmapi)
endif (APPLE)

qt5_use_modules(${PROJECT_NAME} Widgets Gui Network)

add_subdirectory(cryptonote/external)
#add_subdirectory(cryptonote/src)
