cmake_minimum_required(VERSION 3.12)

# Vita toolchain must be selected before project()
if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(
      CMAKE_TOOLCHAIN_FILE
      "$ENV{VITASDK}/share/vita.toolchain.cmake"
      CACHE PATH "toolchain file"
    )
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

project(alex4 C CXX)

include("$ENV{VITASDK}/share/vita.cmake" REQUIRED)

# ---------------------------------------------------------------------------
# Vita application configuration
# ---------------------------------------------------------------------------

set(VITA_APP_NAME "Alex 4")
set(VITA_TITLEID  "WLOGICAX4")
set(VITA_VERSION  "01.00")

add_compile_definitions(
  DATA_DIR="ux0:/data/alex4/"
  ALEX4_CUSTOM_MAPS_ROOT="ux0:data/alex4/custommaps"
  VERSION=1.0
  REVISION=0
  SCREEN_WIDTH=960
  SCREEN_HEIGHT=544
  ALEX4_VITA=1
  fmul=fixmul
  fsin=fixsin
  fcos=fixcos
)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O2")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(VITA_MKSFOEX_FLAGS
  "${VITA_MKSFOEX_FLAGS} -d PARENTAL_LEVEL=1"
)

set(VITA_PREFIX "$ENV{VITASDK}/arm-vita-eabi")

include_directories(
  "${VITA_PREFIX}/include"
)

link_directories(
  "${VITA_PREFIX}/lib"
  ${CMAKE_CURRENT_BINARY_DIR}
)

find_library(
  ALLEGRO_VITA_LIBRARY
  NAMES alleg
  PATHS "${VITA_PREFIX}/lib"
  NO_DEFAULT_PATH
)

find_library(
  ALLEGRO_MAIN_VITA_LIBRARY
  NAMES alleg-main
  PATHS "${VITA_PREFIX}/lib"
  NO_DEFAULT_PATH
)

if(NOT ALLEGRO_VITA_LIBRARY)
  message(FATAL_ERROR
    "Could not find liballeg.a in ${VITA_PREFIX}/lib. "
    "Install the Vita Allegro 4 port first."
  )
endif()

if(NOT ALLEGRO_MAIN_VITA_LIBRARY)
  message(FATAL_ERROR
    "Could not find liballeg-main.a in ${VITA_PREFIX}/lib. "
    "Install the Vita Allegro 4 port first."
  )
endif()

add_library(allegro STATIC IMPORTED GLOBAL)

set_target_properties(
  allegro
  PROPERTIES
    IMPORTED_LOCATION "${ALLEGRO_VITA_LIBRARY}"
    INTERFACE_INCLUDE_DIRECTORIES "${VITA_PREFIX}/include"
)

add_library(alleg-main STATIC IMPORTED GLOBAL)

set_target_properties(
  alleg-main
  PROPERTIES
    IMPORTED_LOCATION "${ALLEGRO_MAIN_VITA_LIBRARY}"
)

set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(BUILD_ALLEGRO4 ON CACHE BOOL "" FORCE)
set(USE_SSE OFF CACHE BOOL "" FORCE)

set(ALLEGRO_VITA 1)

add_subdirectory(third_party/dumb)

set(SOURCE_FILES
  src/actor.c
  src/bullet.c
  src/control.c
  src/edit.c
  src/hisc.c
  src/main.c
  src/map.c
  src/options.c
  src/particle.c
  src/player.c
  src/script.c
  src/scroller.c
  src/shooter.c
  src/timer.c
  src/token.c
)

add_executable(${PROJECT_NAME} ${SOURCE_FILES})

target_include_directories(
  ${PROJECT_NAME}
  PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/src"
    "${CMAKE_CURRENT_SOURCE_DIR}/data"
    "${CMAKE_CURRENT_SOURCE_DIR}/third_party/dumb/include"
    "${VITA_PREFIX}/include"
)

target_compile_options(
  ${PROJECT_NAME}
  PRIVATE
    -fcommon
)

target_link_libraries(
  ${PROJECT_NAME}
  aldmb
  dumb
  allegro
  alleg-main
  vita2d
  SceDisplay_stub
  SceCtrl_stub
  SceAudio_stub
  SceSysmodule_stub
  SceMotion_stub
  SceGxm_stub
  SceCommonDialog_stub
  SceTouch_stub
  SceHid_stub
  SceAudioIn_stub
  SceAppMgr_stub
  SceRtc_stub
  SceLibKernel_stub
  m
  z
)

vita_create_self(
  ${PROJECT_NAME}.self
  ${PROJECT_NAME}
)

vita_create_vpk(
  ${PROJECT_NAME}.vpk
  ${VITA_TITLEID}
  ${PROJECT_NAME}.self

  VERSION ${VITA_VERSION}
  NAME ${VITA_APP_NAME}

  FILE psp2/sce_sys/icon0.png
       sce_sys/icon0.png

  FILE psp2/sce_sys/livearea/contents/bg.png
       sce_sys/livearea/contents/bg.png

  FILE psp2/sce_sys/livearea/contents/startup.png
       sce_sys/livearea/contents/startup.png

  FILE psp2/sce_sys/livearea/contents/template.xml
       sce_sys/livearea/contents/template.xml
)