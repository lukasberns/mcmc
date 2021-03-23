cmake_minimum_required(VERSION 3.6)

set(PROJECT_NAME mcmc)
set(PROJECT_VERSION 1.0)

cmake_policy(SET CMP0048 NEW) # version in project()
project(${PROJECT_NAME} VERSION ${PROJECT_VERSION})


include(PackageBuilder)

pbuilder_prepare_project()

set(CMAKE_CXX_STANDARD 14)

# download some files for MCMC
execute_process(
        COMMAND ./configure -i ${CMAKE_INSTALL_PREFIX}
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
)

set(MCMC_SRC_DIRECTORY ".")

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)
add_compile_options(-Wall -O3 -march=native -ffp-contract=fast -flto -DNDEBUG -DARMA_NO_DEBUG)

include_directories(BEFORE ${MCMC_SRC_DIRECTORY}/include/armodillo_bits)
include_directories(BEFORE ${MCMC_SRC_DIRECTORY}/include/)
include_directories(BEFORE ${MCMC_SRC_DIRECTORY}/include/stats)

MESSAGE (STATUS ${BLAS_LIBRARIES})
set(PACKAGE_LIBS
#        ${BLAS_LIBRARIES}
        )

set(headers
        ${MCMC_SRC_DIRECTORY}/include/aees.hpp
        ${MCMC_SRC_DIRECTORY}/include/armadillo
        ${MCMC_SRC_DIRECTORY}/include/de.hpp
        ${MCMC_SRC_DIRECTORY}/include/determine_bounds_type.hpp
        ${MCMC_SRC_DIRECTORY}/include/hmc.hpp
        ${MCMC_SRC_DIRECTORY}/include/inv_jacobian_adjust.hpp
        ${MCMC_SRC_DIRECTORY}/include/log_jacobian.hpp
        ${MCMC_SRC_DIRECTORY}/include/mala.hpp
        ${MCMC_SRC_DIRECTORY}/include/mcmc.hpp
        ${MCMC_SRC_DIRECTORY}/include/mcmc_options.hpp
        ${MCMC_SRC_DIRECTORY}/include/mcmc_structs.hpp
        ${MCMC_SRC_DIRECTORY}/include/rmhmc.hpp
        ${MCMC_SRC_DIRECTORY}/include/rwmh.hpp
        ${MCMC_SRC_DIRECTORY}/include/transform_vals.hpp

        ${MCMC_SRC_DIRECTORY}/include/stats/dmvnorm.hpp
        ${MCMC_SRC_DIRECTORY}/include/stats/dmvnorm.ipp
        ${MCMC_SRC_DIRECTORY}/include/stats/dnorm.hpp
        ${MCMC_SRC_DIRECTORY}/include/stats/dnorm.ipp
        )


set(sources
        ${MCMC_SRC_DIRECTORY}/src/aees.cpp
        ${MCMC_SRC_DIRECTORY}/src/de.cpp
        ${MCMC_SRC_DIRECTORY}/src/hmc.cpp
        ${MCMC_SRC_DIRECTORY}/src/rmhmc.cpp
        ${MCMC_SRC_DIRECTORY}/src/mala.cpp
        ${MCMC_SRC_DIRECTORY}/src/rwmh.cpp
        )

#pbuilder_library( mcmc SOURCE_FILES DEP_LIBRARIES )
set(PUBLIC_EXT_LIBS ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES}  )

MESSAGE( STATUS ${PUBLIC_EXT_LIBS})
MESSAGE( STATUS ${PRIVATE_EXT_LIBS})
MESSAGE( STATUS ${PACKAGE_LIBS})
pbuilder_library(
        TARGET mcmc
        SOURCES ${sources}
        PROJECT_LIBRARIES ${PACKAGE_LIBS}
        PUBLIC_EXTERNAL_LIBRARIES ${PUBLIC_EXT_LIBS}
        PRIVATE_EXTERNAL_LIBRARIES ${PRIVATE_EXT_LIBS}
)
pbuilder_component_install_and_export(
        COMPONENT Library
        LIBTARGETS mcmc
)
pbuilder_install_headers(${headers})

configure_file(${PROJECT_SOURCE_DIR}/mcmcConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/mcmcConfig.cmake @ONLY)

pbuilder_do_package_config()

#add_executable(test_inside ${SOURCE_FILES})
#set(SOURCE_FILES library/externals/bifrost/test_outside.cxx)
#add_executable(test_outside ${SOURCE_FILES})

#install(TARGETS test_inside test_outside RUNTIME DESTINATION ${BIN_INSTALL_DIR})

