cmake_minimum_required(VERSION 3.10)

project(luajit)

if(WIN32)
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_C_STANDARD 99)
    set(SRC ${CMAKE_CURRENT_SOURCE_DIR}/src)

    include_directories(
        ${SRC}
        ./dynasm
        ${SRC}/host)

    add_executable(minilua ${SRC}/host/minilua.c)

    file(GLOB VM_SOURCES "${SRC}/host/buildvm*.c")
    add_executable(buildvm ${VM_SOURCES})

    add_custom_command(TARGET buildvm
        PRE_BUILD
        COMMAND ./Debug/minilua ${SRC}/../dynasm/dynasm.lua -LN -D WIN -D JIT -D FFI -D P64 -o ${SRC}/host/buildvm_arch.h ${SRC}/vm_x86.dasc
        BYPRODUCTS ${SRC}/host/buildvm_arch.h)
    add_dependencies(buildvm minilua)

    set(ALL_LIB
        ${SRC}/lib_base.c ${SRC}/lib_math.c ${SRC}/lib_bit.c ${SRC}/lib_string.c ${SRC}/lib_table.c ${SRC}/lib_io.c ${SRC}/lib_os.c ${SRC}/lib_package.c ${SRC}/lib_debug.c ${SRC}/lib_jit.c ${SRC}/lib_ffi.c)
    file(GLOB SOURCES ${SRC}/lj_*.c ${SRC}/lib_*.c)
    add_library(luajit SHARED ${SOURCES} ${SRC}/lj_vm.obj ${SRC}/lj_bcdef.h ${SRC}/lj_ffdef.h ${SRC}/lj_libdef.h ${SRC}/lj_recdef.h ${SRC}/jit/vmdef.lua ${SRC}/lj_folddef.h)

    target_compile_definitions(luajit
        PRIVATE _CRT_SECURE_NO_DEPRECATE
        PRIVATE LUA_BUILD_AS_DLL)
    add_dependencies(luajit buildvm)

    add_custom_command(TARGET luajit
        PRE_BUILD
        COMMAND buildvm -m peobj -o ${SRC}/lj_vm.obj
        COMMAND buildvm -m bcdef -o ${SRC}/lj_bcdef.h ${ALL_LIB}
        COMMAND buildvm -m ffdef -o ${SRC}/lj_ffdef.h ${ALL_LIB}
        COMMAND buildvm -m libdef -o ${SRC}/lj_libdef.h ${ALL_LIB}
        COMMAND buildvm -m recdef -o ${SRC}/lj_recdef.h ${ALL_LIB}
        COMMAND buildvm -m vmdef -o ${SRC}/jit/vmdef.lua ${ALL_LIB}
        COMMAND buildvm -m folddef -o ${SRC}/lj_folddef.h ${SRC}/lj_opt_fold.c
        BYPRODUCTS ${SRC}/lj_vm.obj ${SRC}/lj_bcdef.h ${SRC}/lj_ffdef.h ${SRC}/lj_libdef.h ${SRC}/lj_recdef.h ${SRC}/jit/vmdef.lua ${SRC}/lj_folddef.h
    )
    add_custom_command(TARGET luajit POST_BUILD
        COMMAND robocopy $(OutputPath) "D:/Program Files (x86)/Steam/steamapps/common/Don't Starve Together/bin64" luajit.dll \n if ErrorLevel 8 exit /B 1 \n exit /B 0
    )

    # add_executable(LuaJIT_EXE ${SRC}/luajit.c)
    # add_dependencies(LuaJIT_EXE luajit)
endif()
