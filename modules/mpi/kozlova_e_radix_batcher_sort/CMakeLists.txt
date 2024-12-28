get_filename_component(ProjectId ${CMAKE_CURRENT_SOURCE_DIR} NAME)

set(LATEX_OUTPUT_PATH "${CMAKE_BINARY_DIR}/bin")
if (NOT EXISTS ${LATEX_OUTPUT_PATH})
    file(MAKE_DIRECTORY ${LATEX_OUTPUT_PATH})
endif ()

if (USE_LATEX)
    message( STATUS "-- " ${ProjectId} )
    file(GLOB_RECURSE report_files "*.tex")

    foreach (report ${report_files})
        get_filename_component(report_name ${report} NAME_WE)
        list(APPEND list_report_names ${report_name})
    endforeach ()

    add_custom_target( ${ProjectId}_prebuild
            COMMAND ${PDFLATEX_COMPILER} -draftmode -interaction=nonstopmode ${report_files}
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
            DEPENDS ${report_files})

    add_custom_target( ${ProjectId}_pdf
            COMMAND ${PDFLATEX_COMPILER} ${report_files}
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
            DEPENDS  ${report_files})

    add_custom_target(${ProjectId}_all_formats ALL)
    add_dependencies(${ProjectId}_all_formats ${ProjectId}_pdf)

    foreach (report_name ${list_report_names})
            add_custom_command(
                    TARGET ${ProjectId}_all_formats
                    POST_BUILD
                    COMMAND mv "${CMAKE_CURRENT_SOURCE_DIR}/${report_name}.aux" "${LATEX_OUTPUT_PATH}/${report_name}.aux"
                    COMMAND mv "${CMAKE_CURRENT_SOURCE_DIR}/${report_name}.log" "${LATEX_OUTPUT_PATH}/${report_name}.log"
                    COMMAND mv "${CMAKE_CURRENT_SOURCE_DIR}/${report_name}.pdf" "${LATEX_OUTPUT_PATH}/${report_name}.pdf"
            )
    endforeach ()
else()
    message( STATUS "-- ${ProjectId} - NOT BUILD!"  )
endif()
