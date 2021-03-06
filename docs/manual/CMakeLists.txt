macro(qt5_build_doc _qdocconf_name)
  #compute the qdoc template dir from where the qt-html-templates-offline.qdocconf was found
  get_filename_component(QDOC_TEMPLATE_DIR ${QDOC_TEMPLATE} DIRECTORY)

  #compute the qdoc index dir from where the qtcore.index was found
  get_filename_component(QDOC_INDEX_DIR ${QDOC_QTCORE_INDEX} DIRECTORY)
  get_filename_component(QDOC_INDEX_DIR ${QDOC_INDEX_DIR} DIRECTORY)

  # pre-process .qdocconf
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${_qdocconf_name}.qdocconf.in ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}.qdocconf)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${_qdocconf_name}-offline.qdocconf.in ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}-offline.qdocconf)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${_qdocconf_name}-online.qdocconf.in ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}-online.qdocconf)

  # offline docs
  # run qdoc to get qhp file
  set(_qdoc_output_dir ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name})
  # FIXME: this should rather be taken from the qdocconf file?
  file(GLOB _qdoc_srcs ${CMAKE_CURRENT_SOURCE_DIR} "*.qdoc")
  file(GLOB _qdoc_imgs ${CMAKE_CURRENT_SOURCE_DIR}/images "*.png")
  add_custom_command(
    OUTPUT ${_qdoc_output_dir}/${_qdocconf_name}.qhp
    COMMAND ${QDOC_EXECUTABLE}
      --indexdir ${QDOC_INDEX_DIR}
      --outputdir ${_qdoc_output_dir}
    ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}-offline.qdocconf
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}-offline.qdocconf ${_qdoc_srcs} ${_qdoc_imgs}
  )

  # generate qch file from qhp
  add_custom_command(
    OUTPUT ${_qdoc_output_dir}/${_qdocconf_name}.qch
    COMMAND ${QHELPGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}/${_qdocconf_name}.qhp
    DEPENDS ${_qdoc_output_dir}/${_qdocconf_name}.qhp
  )
  add_custom_target(${_qdocconf_name}.qch ALL DEPENDS ${_qdoc_output_dir}/${_qdocconf_name}.qch)

  # install qch
  install(FILES ${_qdoc_output_dir}/${_qdocconf_name}.qch DESTINATION ${QCH_INSTALL_DIR})

  # online docs
  set(_qdoc_output_dir ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}-online)
  add_custom_command(
    OUTPUT ${_qdoc_output_dir}/index.html
    COMMAND ${QDOC_EXECUTABLE}
      --indexdir ${QDOC_INDEX_DIR}
      --outputdir ${_qdoc_output_dir}
    ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}-online.qdocconf
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${_qdocconf_name}-online.qdocconf ${_qdoc_srcs} ${_qdoc_imgs}
  )
  add_custom_target(online-docs DEPENDS ${_qdoc_output_dir}/index.html)

endmacro()

qt5_build_doc(gammaray-manual)
