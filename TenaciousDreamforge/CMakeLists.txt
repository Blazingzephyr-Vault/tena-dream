# Use the version required by DiligentEngine.
cmake_minimum_required(VERSION 3.6)

# Project file name, e.g. VS solution name.
project(TenaciousDreamforge CXX)

# Create a custom target to run source code formatting validation command.
add_format_validation_target(TenaciousDreamforge "${CMAKE_CURRENT_SOURCE_DIR}" TenaciousDreamforge)

# Recursively get source files.
file(GLOB_RECURSE SOURCES "*.c*" "*.h*")

# Add source files to library.
add_library(TenaciousDreamforge STATIC ${SOURCES})

# Create the source groups for source tree with root at PROJECT_SOURCE_DIR.
source_group(TREE ${PROJECT_SOURCE_DIR} FILES ${SOURCES})

# Set target properties.
set_common_target_properties(TenaciousDreamforge)

# Set target folder.
# Set C++20 standard.
set_target_properties(TenaciousDreamforge PROPERTIES
	FOLDER TenaciousDreamforge
	CXX_STANDARD 20
	CXX_STANDARD_REQUIRED ON
)

if(MSVC)
	# Enable Unicode support.
	# Disable w4201: nonstandard extension used: nameless struct/union.
	target_compile_options(TenaciousDreamforge PRIVATE -DUNICODE /wd4201)

	if(PLATFORM_UNIVERSAL_WINDOWS)
        # Disable w4189: local variable is initialized but not referenced.
        # Disable w4063: case is not a valid value for switch of enum.
        # Consume the windows runtime extensions (/ZW).
		target_compile_options(TenaciousDreamforge INTERFACE /wd4189 /wd4063 /ZW)
	endif()
endif()

# Enumerate engine header subdirectories.
file(GLOB_RECURSE HEADER_DIRS LIST_DIRECTORIES true RELATIVE ${CMAKE_CURRENT_LIST_DIR} "*")
list(FILTER HEADER_DIRS INCLUDE REGEX ".*(include|interface)$")

# Include headers.
target_include_directories(TenaciousDreamforge PUBLIC ${HEADER_DIRS})

# Get supported backend targets.
get_supported_backends(ENGINE_LIBRARIES)

# Link libraries.
target_link_libraries(TenaciousDreamforge
PRIVATE 
    Diligent-BuildSettings
PUBLIC
    Diligent-Common
    Diligent-GraphicsTools
    Diligent-TextureLoader
    Diligent-TargetPlatform
    Diligent-Imgui
    Diligent-GraphicsAccessories
    ${ENGINE_LIBRARIES}
)

if(PLATFORM_UNIVERSAL_WINDOWS)
    target_link_libraries(TenaciousDreamforge PRIVATE dxguid.lib)
elseif(PLATFORM_ANDROID)
    target_link_libraries(TenaciousDreamforge PRIVATE GLESv3 PUBLIC native_app_glue)
elseif(PLATFORM_LINUX)
    target_link_libraries(TenaciousDreamforge PRIVATE XCBKeySyms GL X11)
endif()

if(PLATFORM_WIN32)
	# Copy API libraries.
	copy_required_dlls(TenaciousDreamforge)
endif()

if(PLATFORM_MACOS AND VULKAN_LIB_PATH)
	# Configure rpath so that library can find Vulkan library.
	set_target_properties(TenaciousDreamforge PROPERTIES
		BUILD_RPATH "${VULKAN_LIB_PATH}"
	)
endif()
