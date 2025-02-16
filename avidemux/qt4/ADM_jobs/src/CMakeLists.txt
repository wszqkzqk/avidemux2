##########################################
# Config
##########################################
ADD_DEFINITIONS(-DADM_UI_TYPE_BUILD=ADM_UI_QT4)

IF(MSVC)
ADD_DEFINITIONS(-D_WINSOCKAPI_)
ENDIF(MSVC)

SET(CONFIG_HEADER_TYPE ADM_BUILD_QT4)
SET(UI_SUFFIX qt4)
include_directories(../include ${QT_INCLUDE_DIR} ${CMAKE_CURRENT_BINARY_DIR})
INCLUDE(admWindRes)
###########################################
# Executable
###########################################
#
SET(UI uiJobs.ui uiProgress.ui)
SET(headers ../include/T_jobs.h ../include/T_progress.h)
IF(APPLE OR WIN32)
    ADM_QT_ADD_RESOURCES(resource jobs.qrc)
ELSE(APPLE OR WIN32)
    ADM_QT_ADD_RESOURCES(resource jobs_linux.qrc)
ENDIF(APPLE OR WIN32)
ADM_QT_WRAP_UI(ADM_JOBS_headers ${UI})
ADM_QT_WRAP_CPP(ADM_JOBS_source ${headers})
SET(ADM_JOB_SRCS 
                ADM_jobs.cpp
                ADM_jobControl.cpp
                ADM_runOneJob.cpp
		ADM_jobProgress.cpp
		../../ADM_userInterfaces/translator.cpp
                ${ADM_JOBS_headers}
                ${ADM_JOBS_source}
               	${resource} 
        )

###########################################
# Version Info
###########################################
if (WIN32)
        WINDRESIFY(mainJobs "${CMAKE_CURRENT_SOURCE_DIR}/../../../common/xpm/adm.ico" src "avidemux_jobs" "Avidemux Jobs")
        SET(ADM_JOB_SRCS ${ADM_JOB_SRCS} ${src})
endif (WIN32)

###########################################
# Executable
###########################################
include(admCoreIncludes)
include_directories("${PTHREAD_INCLUDE_DIR}")

IF(WIN32)
    ADD_EXECUTABLE(avidemux3_jobs_${QT_EXTENSION} WIN32 ${ADM_JOB_SRCS})
    IF(MSVC)
        set_target_properties(avidemux3_jobs_${QT_EXTENSION} PROPERTIES LINK_FLAGS_DEBUG "/SUBSYSTEM:WINDOWS")
	    set_target_properties(avidemux3_jobs_${QT_EXTENSION} PROPERTIES WIN32_EXECUTABLE True)
    ELSE(MSVC)
	    target_link_libraries(avidemux3_jobs_${QT_EXTENSION} winmm -mwindows)
    ENDIF(MSVC)
ELSE(WIN32)
    ADD_EXECUTABLE(avidemux3_jobs_${QT_EXTENSION} ${ADM_JOB_SRCS})
ENDIF(WIN32)


ADD_SOURCE_CFLAGS(ADM_jobs.cpp "-DADM_VERSION=\"${AVIDEMUX_VERSION}\"")
if (ADM_SUBVERSION)
        ADD_SOURCE_CFLAGS(ADM_jobs.cpp "-DADM_SUBVERSION=\"${ADM_SUBVERSION}\"")
endif (ADM_SUBVERSION)

###########################################
TARGET_LINK_LIBRARIES(avidemux3_jobs_${QT_EXTENSION} ADM_coreSocket6)
TARGET_LINK_LIBRARIES(avidemux3_jobs_${QT_EXTENSION} ${QT_QTGUI_LIBRARY} ${QT_QTCORE_LIBRARY})
TARGET_LINK_LIBRARIES(avidemux3_jobs_${QT_EXTENSION} ${PTHREAD_LIBRARIES})
###########################################
# OS Specific
###########################################
if (WIN32 OR APPLE)
	set_property(TARGET avidemux3_jobs_${QT_EXTENSION} PROPERTY OUTPUT_NAME avidemux_jobs)
endif (WIN32 OR APPLE)


IF (APPLE)
	add_source_cflags(ADM_runOneJob.cpp "-DAVIDEMUX_MAJOR_MINOR=\"${AVIDEMUX_MAJOR_MINOR}\"")
	target_link_libraries(avidemux3_jobs_${QT_EXTENSION} "-framework CoreServices -framework CoreAudio -framework AudioUnit -framework Carbon")

	# for Leopard but it doesn't hurt Tiger
	target_link_libraries(avidemux3_jobs_${QT_EXTENSION} "-Wl,-dylib_file,/System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGL.dylib:/System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGL.dylib")
ENDIF (APPLE)

TARGET_LINK_LIBRARIES(avidemux3_jobs_${QT_EXTENSION} ADM_coreJobs)
TARGET_LINK_LIBRARIES(avidemux3_jobs_${QT_EXTENSION} ADM_coreUtils6)
TARGET_LINK_LIBRARIES(avidemux3_jobs_${QT_EXTENSION} ADM_core6)
#
target_link_libraries(avidemux3_jobs_${QT_EXTENSION} "-lm -lstdc++")
#
#


###########################################
# Install
###########################################
ADM_LINK_THREAD(avidemux3_jobs_${QT_EXTENSION})
ADM_INSTALL_BIN(avidemux3_jobs_${QT_EXTENSION})
#
# Packaging
#
#include(admPackager)
#admPackager(qt4Package)
