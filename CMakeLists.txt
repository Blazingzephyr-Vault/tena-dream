# Use the version required by DiligentEngine.
cmake_minimum_required(VERSION 3.6)

# Project file name, e.g. VS solution name.
project(TenaciousDreamforge CXX)

# Sort the target objects into folders.
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Build DiligentEngine.
add_subdirectory(DiligentCore)
add_subdirectory(DiligentTools)

# Build TenaciousDreamforgeEngine.
add_subdirectory(TenaciousDreamforge)
