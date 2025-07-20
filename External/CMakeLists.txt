# External/CMakeLists.txt
cmake_minimum_required(VERSION 3.16)

# Disable warnings for external libraries
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w")

# ============================================================================
# GLM - Header-only math library
# ============================================================================
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/glm/CMakeLists.txt")
    add_subdirectory(glm)
endif()

# ============================================================================
# GLFW - Window and input handling
# ============================================================================
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/glfw/CMakeLists.txt")
    # Configure GLFW options
    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
    
    add_subdirectory(glfw)
endif()

# ============================================================================
# spdlog - Fast C++ logging library
# ============================================================================
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/spdlog/CMakeLists.txt")
    set(SPDLOG_BUILD_EXAMPLE OFF CACHE BOOL "" FORCE)
    set(SPDLOG_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(SPDLOG_INSTALL OFF CACHE BOOL "" FORCE)
    
    add_subdirectory(spdlog)
endif()

# ============================================================================
# nlohmann/json - JSON library
# ============================================================================
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/json/CMakeLists.txt")
    set(JSON_BuildTests OFF CACHE BOOL "" FORCE)
    set(JSON_Install OFF CACHE BOOL "" FORCE)
    
    add_subdirectory(json)
endif()

# ============================================================================
# EnTT - Header-only entity component system
# ============================================================================
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/entt/CMakeLists.txt")
    set(ENTT_BUILD_TESTING OFF CACHE BOOL "" FORCE)
    set(ENTT_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(ENTT_BUILD_EXAMPLE OFF CACHE BOOL "" FORCE)
    
    add_subdirectory(entt)
endif()

# ============================================================================
# Vulkan/SPIR-V Tools (ordre important pour les dépendances)
# ============================================================================

# SPIRV-Headers (doit être en premier)
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/SPIRV-Headers/CMakeLists.txt")
    add_subdirectory(SPIRV-Headers)
endif()

# SPIRV-Tools (dépend de SPIRV-Headers)
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/SPIRV-Tools/CMakeLists.txt")
    set(SPIRV_SKIP_TESTS ON CACHE BOOL "" FORCE)
    set(SPIRV_SKIP_EXECUTABLES ON CACHE BOOL "" FORCE)
    
    add_subdirectory(SPIRV-Tools)
endif()

# glslang (dépend de SPIRV-Tools)
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/glslang/CMakeLists.txt")
    set(ENABLE_SPVREMAPPER OFF CACHE BOOL "" FORCE)
    set(ENABLE_GLSLANG_BINARIES OFF CACHE BOOL "" FORCE)
    set(ENABLE_GLSLANG_JS OFF CACHE BOOL "" FORCE)
    set(ENABLE_RTTI ON CACHE BOOL "" FORCE)
    set(ENABLE_EXCEPTIONS ON CACHE BOOL "" FORCE)
    set(ENABLE_OPT OFF CACHE BOOL "" FORCE)
    
    add_subdirectory(glslang)
endif()

# shaderc (dépend de glslang et SPIRV-Tools)
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/shaderc/CMakeLists.txt")
    set(SHADERC_SKIP_TESTS ON CACHE BOOL "" FORCE)
    set(SHADERC_SKIP_EXAMPLES ON CACHE BOOL "" FORCE)
    set(SHADERC_SKIP_COPYRIGHT_CHECK ON CACHE BOOL "" FORCE)
    
    add_subdirectory(shaderc)
endif()

# ============================================================================
# Créer des alias pour un usage plus facile dans le projet principal
# ============================================================================

# Créer des targets d'alias si elles n'existent pas déjà
if(TARGET glfw AND NOT TARGET External::GLFW)
    add_library(External::GLFW ALIAS glfw)
endif()

if(TARGET spdlog AND NOT TARGET External::spdlog)
    add_library(External::spdlog ALIAS spdlog)
endif()

if(TARGET nlohmann_json AND NOT TARGET External::json)
    add_library(External::json ALIAS nlohmann_json)
endif()

if(TARGET EnTT::EnTT AND NOT TARGET External::EnTT)
    add_library(External::EnTT ALIAS EnTT::EnTT)
endif()

if(TARGET glslang AND NOT TARGET External::glslang)
    add_library(External::glslang ALIAS glslang)
endif()

if(TARGET shaderc AND NOT TARGET External::shaderc)
    add_library(External::shaderc ALIAS shaderc)
endif()

# GLM est header-only, on peut créer une target interface
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/glm/glm" AND NOT TARGET External::GLM)
    add_library(External::GLM INTERFACE)
    target_include_directories(External::GLM INTERFACE glm)
endif()

# Message de confirmation
message(STATUS "External libraries configured successfully")