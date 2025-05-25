find_program(GZIP gzip
  HINTS
  /bin
  /usr/bin
  /usr/local/bin)

if(GZIP-NOTFOUND)
  message(WARNING "Gzip not found! Uncompressed manpage installed")
  add_custom_target(documentation ALL
    DEPENDS vobsub2srt.1)

  install(FILES vobsub2srt.1 DESTINATION ${INSTALL_MAN_DIR})
else()
  add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/doc/vobsub2srt.1.gz
    COMMAND ${GZIP} -9 -c vobsub2srt.1 > ${CMAKE_BINARY_DIR}/doc/vobsub2srt.1.gz
    MAIN_DEPENDENCY vobsub2srt.1
    WORKING_DIRECTORY "${vobsub2srt_SOURCE_DIR}/doc")

  add_custom_target(documentation ALL
    DEPENDS ${CMAKE_BINARY_DIR}/doc/vobsub2srt.1.gz)

  install(FILES ${CMAKE_BINARY_DIR}/doc/vobsub2srt.1.gz DESTINATION ${INSTALL_MAN_DIR})
endif()

#option(BASH_COMPLETION_PATH "Install the bash completion script to the given path.  Should be set to the value of `pkg-config --variable=completionsdir bash-completion` if available.")

if(NOT BASH_COMPLETION_PATH)
  # Instead of calling pkg-config it should be possible to use
  # find_package(bash-completions).  See the bash-completions README.
  # But Debian/Ubuntu don't ship the cmake configs at the moment.
  # Therefore we fallback to pkg-config.
  find_program(PKG_CONFIG_EXECUTABLE NAMES pkg-config DOC "pkg-config executable")
  mark_as_advanced(PKG_CONFIG_EXECUTABLE)
  if(PKG_CONFIG_EXECUTABLE)
    execute_process(COMMAND ${PKG_CONFIG_EXECUTABLE} --variable=completionsdir bash-completion
      OUTPUT_VARIABLE BASH_COMPLETION_PATH
      ERROR_QUIET
      OUTPUT_STRIP_TRAILING_WHITESPACE)
  endif()

  # ————————————————————————————————————————————————————
  # If the user explicitly set CMAKE_INSTALL_PREFIX on the command line,
  # force completions under that prefix/share/bash-completion/completions
  if(NOT CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(BASH_COMPLETION_PATH
      "${CMAKE_INSTALL_PREFIX}/share/bash-completion/completions"
        CACHE PATH
        "Location to install bash completions (forced under PREFIX)" )
    message(STATUS "Overriding Bash completion path to ${BASH_COMPLETION_PATH}")
  endif()
  # ————————————————————————————————————————————————————
endif()

if(BASH_COMPLETION_PATH)
  message(STATUS "Bash completion path: ${BASH_COMPLETION_PATH}")
  install(FILES completion.sh DESTINATION ${BASH_COMPLETION_PATH} RENAME vobsub2srt)
else()
  message(STATUS "Bash completions not installed (set -DBASH_COMPLETION_PATH)")
endif()
