#/* +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#   Copyright (c) 2024 The plumed team
#   (see the PEOPLE file at the root of the distribution for a list of names)
#
#   See http://www.plumed.org for more information.
#
#   This file is part of plumed, version 2.
#
#   plumed is free software: you can redistribute it and/or modify
#   it under the terms of the GNU Lesser General Public License as published by
#   the Free Software Foundation, either version 3 of the License, or
#   (at your option) any later version.
#
#   plumed is distributed in the hope that it will be useful,
#   but WITHOUT ANY WARRANTY; without even the implied warranty of
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#   GNU Lesser General Public License for more details.
#
#   You should have received a copy of the GNU Lesser General Public License
#   along with plumed.  If not, see <http://www.gnu.org/licenses/>.
#+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ */

cmake_minimum_required(VERSION 3.15...3.26)
project(${SKBUILD_PROJECT_NAME} LANGUAGES C)

find_package(
  Python
  COMPONENTS Interpreter Development.Module
  REQUIRED)

add_custom_command(
  OUTPUT plumed.c
  COMMENT
    "Making ${CMAKE_CURRENT_BINARY_DIR}/plumed.c from ${CMAKE_CURRENT_SOURCE_DIR}/plumed.pyx"
  COMMAND Python::Interpreter -m cython
          "${CMAKE_CURRENT_SOURCE_DIR}/plumed.pyx" --output-file plumed.c
  DEPENDS plumed.pyx
  VERBATIM)

python_add_library(plumed MODULE plumed.c WITH_SOABI)
target_include_directories(plumed PRIVATE include)
target_compile_definitions(plumed PRIVATE 
                        __PLUMED_HAS_DLOPEN
                        __PLUMED_WRAPPER_LINK_RUNTIME=1
                        __PLUMED_WRAPPER_IMPLEMENTATION=1
                        __PLUMED_WRAPPER_EXTERN=0
                        #__PLUMED_WRAPPER_ENABLE_RTLD_DEEPBIND=0
)

install(TARGETS plumed DESTINATION .)