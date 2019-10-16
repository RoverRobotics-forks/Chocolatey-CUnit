cmake_minimum_required(VERSION 3.5)
project(choco-CUnit LANGUAGES C)

include(ExternalProject)

set(nupkg_staging "${CMAKE_CURRENT_BINARY_DIR}/nupkg_staging")
ExternalProject_Add(CUnit
  GIT_REPOSITORY "https://github.com/RoverRobotics-forks/CUnit.git"
  PREFIX "CUNIT-PREFIX"
  CMAKE_ARGS
      "-DCUNIT_CMAKE_INSTALL=yes"
      "-DCMAKE_INSTALL_PREFIX=${nupkg_staging}"
)

# paths in the nuspec are relative to its location so we need to copy everything there
file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/tools" "${CMAKE_CURRENT_SOURCE_DIR}/cunit.nuspec" DESTINATION "${nupkg_staging}")


set(cunit_version "2.1.3")
set(cunit_nupkg "cunit.${cunit_version}.nupkg")
add_custom_command(OUTPUT "${cunit_nupkg}"
	COMMAND choco ARGS pack --yes "--out" "${CMAKE_CURRENT_BINARY_DIR}" "--version=${cunit_version}" "cunit.nuspec"
	DEPENDS cunit.nuspec CUnit
	WORKING_DIRECTORY "${nupkg_staging}"
	VERBATIM)

add_custom_target("choco" ALL DEPENDS "${cunit_nupkg}")

install(CODE "execute_process(COMMAND choco install --yes --force \"${CMAKE_CURRENT_BINARY_DIR}/${cunit_nupkg}\")")
