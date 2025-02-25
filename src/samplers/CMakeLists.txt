SET(Samplers
    Whitenoise
    Stratified
    HexagonalGrid
    RegularGrid
    Rank1
    Halton
    Hammersley
    Faure
    NRooks
    Niederreiter
    Korobov
    Kronecker
    DartThrowing
    Bridson
    AAPatterns
    ART
    CMJ
    FastPoisson
    Fattal
    ForcedRandom
    VanDerCorput
    Step
    SinglePeak
    Penrose
    ProjectiveBlueNoise
    PMJ
    LDBN
    LutLDBN
    Sobol
)

SET(pyutk_samplers)
SET(pyutk_samplers_libs pybind11::module spdlog::spdlog)

FOREACH(FILE ${Samplers})
    IF(UTK_BUILD_EXE)
        add_executable(${UTK_EXE_PREFIX}${FILE} ${FILE}.cpp)
        target_include_directories(${UTK_EXE_PREFIX}${FILE} PRIVATE "${UTK_INCLUDE_DIR}")
    ENDIF()

    IF (EXISTS ${UTK_EXTERNALS_FOLDER}/${FILE})
        FILE(GLOB_RECURSE ADDITIONNAL_CODE
                    "${UTK_EXTERNALS_FOLDER}/${FILE}/*.cpp"
            )
        
        LIST(APPEND pyutk_samplers ${ADDITIONNAL_CODE})
        
        IF (UTK_BUILD_EXE)
            target_sources(${UTK_EXE_PREFIX}${FILE} PRIVATE ${ADDITIONNAL_CODE})
            target_include_directories(${UTK_EXE_PREFIX}${FILE} PRIVATE ${UTK_EXTERNALS_FOLDER})
        ENDIF()
    ENDIF()

    IF(UTK_BUILD_EXE)
        target_link_libraries(${UTK_EXE_PREFIX}${FILE} PRIVATE spdlog::spdlog CLI11::CLI11)

        install(TARGETS ${UTK_EXE_PREFIX}${FILE}
            RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib)
    ENDIF()
ENDFOREACH(FILE)

# ===========================================
#  Samplers requiring additional libraries  #
# ===========================================

# Disable compilation on windows... GEOGRAM dllspec is problematic for an unknown reason...
IF(NOT WIN32)
    IF(OpenMP_FOUND)
        IF(UTK_BUILD_EXE)
            # Lloyd
            add_executable(${UTK_EXE_PREFIX}Lloyd Lloyd.cpp  ${UTK_EXTERNALS_FOLDER}/Lloyd/Delaunay_psm.cpp)
            
            target_include_directories(${UTK_EXE_PREFIX}Lloyd PRIVATE "${UTK_INCLUDE_DIR}")
            target_include_directories(${UTK_EXE_PREFIX}Lloyd PRIVATE "${UTK_EXTERNALS_FOLDER}/")
            target_link_libraries(${UTK_EXE_PREFIX}Lloyd PRIVATE spdlog::spdlog CLI11::CLI11 OpenMP::OpenMP_CXX)

            install(TARGETS ${UTK_EXE_PREFIX}Lloyd
                RUNTIME DESTINATION bin
                LIBRARY DESTINATION lib
                ARCHIVE DESTINATION lib)
        ENDIF()
        
        LIST(APPEND pyutk_samplers_libs OpenMP::OpenMP_CXX)
        LIST(APPEND pyutk_samplers ${UTK_EXTERNALS_FOLDER}/Lloyd/Delaunay_psm.cpp)
        add_definitions(-DPYUTK_LLOYD)
    ELSE()
        # message("Lloyd not compiled.")
    ENDIF()
ELSE()
    message("Lloyd not compiled on Windows !")
ENDIF()

# BNOT
IF (CGAL_FOUND AND SuiteSparse_FOUND)
    IF(UTK_BUILD_EXE)
        add_executable(${UTK_EXE_PREFIX}BoundedBNOT  BoundedBNOT.cpp)
        add_executable(${UTK_EXE_PREFIX}PeriodicBNOT PeriodicBNOT.cpp)

        target_include_directories(${UTK_EXE_PREFIX}BoundedBNOT PRIVATE "${UTK_INCLUDE_DIR}")
        target_include_directories(${UTK_EXE_PREFIX}BoundedBNOT PRIVATE "${UTK_EXTERNALS_FOLDER}/")
        target_link_libraries(${UTK_EXE_PREFIX}BoundedBNOT PRIVATE spdlog::spdlog CLI11::CLI11)
        target_link_libraries(${UTK_EXE_PREFIX}BoundedBNOT PRIVATE CGAL::CGAL SuiteSparse::AMD SuiteSparse::CHOLMOD SuiteSparse::SPQR)        

        target_include_directories(${UTK_EXE_PREFIX}PeriodicBNOT PRIVATE "${UTK_INCLUDE_DIR}")
        target_include_directories(${UTK_EXE_PREFIX}PeriodicBNOT PRIVATE "${UTK_EXTERNALS_FOLDER}/")
        target_link_libraries(${UTK_EXE_PREFIX}PeriodicBNOT PRIVATE spdlog::spdlog CLI11::CLI11)
        target_link_libraries(${UTK_EXE_PREFIX}PeriodicBNOT PRIVATE CGAL::CGAL SuiteSparse::AMD SuiteSparse::CHOLMOD SuiteSparse::SPQR)
    
        
        install(TARGETS ${UTK_EXE_PREFIX}BoundedBNOT
            RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib)

        install(TARGETS ${UTK_EXE_PREFIX}PeriodicBNOT
            RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib)
    ENDIF()
    
    LIST(APPEND pyutk_samplers_libs CGAL::CGAL SuiteSparse::AMD SuiteSparse::CHOLMOD SuiteSparse::SPQR)
    add_definitions(-DPYUTK_BNOT)
ELSE()
    # message("BNOT Not compiled")
ENDIF()

IF (UTK_PYTHON)
    SET(pyutk_samplers_bindings 
        "python/Samplers.cpp"
        "python/BaseSamplers.cpp"
        "python/ExternalSamplers.cpp"
    )
    
    add_library(pyutkSamplers ${pyutk_samplers} ${pyutk_samplers_bindings})

    # Required by pybind
    set_target_properties(pyutkSamplers PROPERTIES POSITION_INDEPENDENT_CODE ON) 
    
    target_include_directories(pyutkSamplers PRIVATE "${UTK_INCLUDE_DIR}")
    target_include_directories(pyutkSamplers PRIVATE ${UTK_EXTERNALS_FOLDER})
    target_link_libraries(pyutkSamplers PRIVATE ${pyutk_samplers_libs})
ENDIF()
