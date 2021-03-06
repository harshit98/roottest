if(ROOTTEST_DIR)
      set(ROOT_EVENT_DIR ${ROOTTEST_DIR}/root/treeformula/event/)
else()
      set(ROOT_EVENT_DIR ${ROOT_SOURCE_DIR}/roottest/root/treeformula/event/)
endif()

if(TARGET onepcm)
      set(EventDependencies "onepcm")
endif()

# Generating dataset from tree-treeformula-event test
# FIXME: it will be nice to move  tree-treeformula-event to CMake and add it as dependency
ROOT_GENERATE_DICTIONARY(EventDict ${ROOT_EVENT_DIR}/Event.h 
                  LINKDEF ${ROOT_EVENT_DIR}/EventLinkDef.h)

ROOT_LINKER_LIBRARY(EventTreeFormula TEST ${ROOT_EVENT_DIR}/Event.cxx EventDict.cxx
                  LIBRARIES Core Tree Hist MathCore)

ROOTTEST_GENERATE_EXECUTABLE(EventGenerate ${ROOT_EVENT_DIR}/MainEvent.cxx 
                  LIBRARIES Core RIO Net Tree Hist MathCore EventTreeFormula)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Event.root
                  COMMAND ./EventGenerate 6 0 0 1 30 > log
                  DEPENDS EventGenerate ${EventDependencies})
add_custom_target(event-generation ALL DEPENDS Event.root)

add_custom_command(
        TARGET event-generation
        COMMAND ${CMAKE_COMMAND} -E copy
                Event.root
                event1.root
        COMMAND ${CMAKE_COMMAND} -E copy
                Event.root
                event2.root)


if(ROOTTEST_DIR)
      set(ROOT_DIR ${ROOTSYS})
else()
      set(ROOT_DIR ${ROOT_SOURCE_DIR})
endif()

add_custom_target(hsimple-file ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/hsimple.root)

if(TARGET hsimple)
      set(HSimpleDependencies "hsimple")
      add_dependencies(hsimple-file ${HSimpleDependencies})
endif()

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/hsimple.root
                  COMMAND ${ROOT_root_CMD} -q -l -b ${ROOT_DIR}/tutorials/hsimple.C -e "{ TFile f(\"hsimple.root\"); TTree *ntuple; f.GetObject(\"ntuple\",ntuple); return ntuple ? 0 : 1; }" > hsimple.log
                  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                  DEPENDS ${HSimpleDependencies}
                  VERBATIM)

add_custom_command(
        TARGET  hsimple-file
        COMMAND ${CMAKE_COMMAND} -E copy
                ${CMAKE_CURRENT_BINARY_DIR}/hsimple.root
                hsimple1.root
        COMMAND ${CMAKE_COMMAND} -E copy
                ${CMAKE_CURRENT_BINARY_DIR}/hsimple.root
                hsimple2.root)

set(RootExeOptions -e "gSystem->Load(\"libEventTreeFormula\")")

ROOTTEST_ADD_TEST(treeCloneTest
                  MACRO runEvent.C
                  PRECMD ${ROOT_root_CMD} -b -q -l ${CMAKE_CURRENT_SOURCE_DIR}/run.C
                  OUTREF references/treeCloneTest.ref)

ROOTTEST_ADD_TEST(runtreeCloneTest2
                  MACRO  runtreeCloneTest2.C
                  PRECMD cp -r ${CMAKE_CURRENT_SOURCE_DIR}/files/root ${CMAKE_CURRENT_BINARY_DIR}
                  OUTCNVCMD grep -v Adding
                  OUTREF references/treeCloneTest2.ref)
if(compression_default STREQUAL "lz4")
    if(${CMAKE_SIZEOF_VOID_P} EQUAL "8")
            ROOTTEST_ADD_TEST(exectrim
                  MACRO exectrim.C
                  COPY_TO_BUILDDIR files/output_Coulomb_LER_study_10.root
                  OUTREF references/exectrimLZ4.ref)
    else()
            ROOTTEST_ADD_TEST(exectrim
                  MACRO exectrim.C
                  COPY_TO_BUILDDIR files/output_Coulomb_LER_study_10.root
                  OUTREF references/exectrimLZ4_i686.ref)
    endif()
else()
            ROOTTEST_ADD_TEST(exectrim
                  MACRO exectrim.C
                  COPY_TO_BUILDDIR files/output_Coulomb_LER_study_10.root
                  OUTREF references/exectrimZLIB.ref)
endif()

ROOTTEST_ADD_TEST(deepClass
                  MACRO rundeepClass.C
                  PRECMD ${ROOT_root_CMD} -b -q -l ${CMAKE_CURRENT_SOURCE_DIR}/make_CopyTree.C
                  COPY_TO_BUILDDIR files/ver_40200.root
                  OUTREF references/deepClass.ref)

ROOTTEST_ADD_TEST(mergeWithDir
                  MACRO runmergeWithDir.C
                  COPY_TO_BUILDDIR files/file1.root files/file2.root
                  OUTREF references/mergeWithDir.ref)

ROOTTEST_ADD_TEST(index
                  MACRO runindex.C
                  OUTREF references/index.ref)
