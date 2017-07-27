CMAKE_MINIMUM_REQUIRED ( VERSION 2.8 )
    
#   255  ./configure --enable-experimental --enable-module-recovery --enable-module-ecdh --prefix=/home/nyue/systems/secp256k1

# ./contrib/lax_der_parsing.c
# ./contrib/lax_der_privatekey_parsing.c
# ./src/bench_ecdh.c
# ./src/bench_recover.c
# ./src/bench_verify.c
# ./src/tests.c
# ./src/java/org_bitcoin_NativeSecp256k1.c
# ./src/java/org_bitcoin_Secp256k1Context.c
# ./src/gen_context.c
# ./src/tests_exhaustive.c
# ./src/bench_sign.c
# ./src/bench_internal.c
# ./src/secp256k1.c

OPTION ( ENABLE_EXPERIMENTAL "Enable experimental code path" OFF)
OPTION ( ENABLE_MODULE_RECOVERY "Enable recovery module" ON)
OPTION ( ENABLE_MODULE_ECDH "Enable ecdh module" OFF)
OPTION ( ENABLE_JAVA "Enable java" OFF)
OPTION ( ENABLE_SHARED "Enable shared library build" OFF)

IF ( ENABLE_JAVA )
  FIND_PACKAGE ( JNI REQUIRED COMPONENTS Development )
  INCLUDE_DIRECTORIES ( ${JNI_INCLUDE_DIRS} )
ENDIF ()

INCLUDE_DIRECTORIES ( . )

LIST ( APPEND SECP256K1_SOURCES src/gen_context.c )
LIST ( APPEND SECP256K1_SOURCES src/secp256k1.c )

IF ( ENABLE_JAVA )
  LIST ( APPEND SECP256K1_SOURCES src/java/org_bitcoin_NativeSecp256k1.c )
  LIST ( APPEND SECP256K1_SOURCES src/java/org_bitcoin_Secp256k1Context.c )
ENDIF ()

# ADD_DEFINITIONS ( -DHAVE_LIBGMP )
# ADD_DEFINITIONS ( -DUSE_NUM_GMP )
# ADD_DEFINITIONS ( -DUSE_FIELD_INV_NUM )
# ADD_DEFINITIONS ( -DUSE_SCALAR_INV_NUM )
ADD_DEFINITIONS ( -DUSE_NUM_NONE )
# ADD_DEFINITIONS ( -DUSE_FIELD_5X52 ) # 64bit
ADD_DEFINITIONS ( -DUSE_FIELD_10X26 ) # 32bit
ADD_DEFINITIONS ( -DUSE_FIELD_INV_BUILTIN )
ADD_DEFINITIONS ( -DUSE_SCALAR_INV_BUILTIN )
# ADD_DEFINITIONS ( -DUSE_SCALAR_4X64 )
ADD_DEFINITIONS ( -DUSE_SCALAR_8X32 )

IF ( ENABLE_MODULE_RECOVERY )
  ADD_DEFINITIONS ( -DENABLE_MODULE_RECOVERY )
ENDIF ()

IF ( ENABLE_MODULE_ECDH )
  ADD_DEFINITIONS ( -DENABLE_MODULE_ECDH )
ENDIF ()

ADD_LIBRARY ( secp256k1_static STATIC
  ${SECP256K1_SOURCES}
  )

IF ( ENABLE_SHARED )
  ADD_LIBRARY ( secp256k1_shared SHARED
    ${SECP256K1_SOURCES}
    )
ENDIF()

SET_TARGET_PROPERTIES ( secp256k1_static
  PROPERTIES OUTPUT_NAME secp256k1
  )

IF ( ENABLE_SHARED )
  SET_TARGET_PROPERTIES ( secp256k1_shared
    PROPERTIES OUTPUT_NAME secp256k1
    )
ENDIF ()

INSTALL ( TARGETS
  secp256k1_static
  DESTINATION
  ${CMAKE_INSTALL_PREFIX}/lib
  )

IF ( ENABLE_SHARED )
  INSTALL ( TARGETS
    secp256k1_shared
    DESTINATION
    ${CMAKE_INSTALL_PREFIX}/lib
    )
ENDIF ()

INSTALL ( FILES
  include/secp256k1.h
  DESTINATION
  ${CMAKE_INSTALL_PREFIX}/include
  )

IF (ENABLE_MODULE_RECOVERY)
  INSTALL ( FILES
    include/secp256k1_recovery.h
    DESTINATION
    ${CMAKE_INSTALL_PREFIX}/include
    )
ENDIF ()

IF (ENABLE_MODULE_ECDH)
  INSTALL ( FILES
    include/secp256k1_ecdh.h
    DESTINATION
    ${CMAKE_INSTALL_PREFIX}/include
    )
ENDIF ()
