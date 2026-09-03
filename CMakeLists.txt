project(mig)

cmake_minimum_required(VERSION 3.13)

find_package(BISON)
find_package(FLEX)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
if (NOT DARLING_HOST_MACOS)
	add_definitions(-Du_int=unsigned -D__int64_t=int64_t -D__int32_t=int32_t -D__darwin_natural_t=unsigned -D__uint32_t=uint32_t -D__uint16_t=uint16_t -D__uint64_t=uint64_t -D_BSD_I386__TYPES_H_ -D_BSD_ARM__TYPES_H_ -D__arm64__=1)
endif()
add_definitions('-DMIG_VERSION="1.0"')
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/darling/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/darling/include/mach)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/migcom.tproj)

BISON_TARGET(MyParser migcom.tproj/parser.y ${CMAKE_CURRENT_BINARY_DIR}/parser.c)
FLEX_TARGET(MyScanner migcom.tproj/lexxer.l ${CMAKE_CURRENT_BINARY_DIR}/lexer.c)
ADD_FLEX_BISON_DEPENDENCY(MyScanner MyParser)

# Hide some unimportant warnings
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-dangling-else -Wno-implicit-function-declaration")

set(mig_sources
	migcom.tproj/error.c
	migcom.tproj/global.c
	#migcom.tproj/handler.c
	migcom.tproj/header.c
	migcom.tproj/mig.c
	migcom.tproj/routine.c
	migcom.tproj/server.c
	migcom.tproj/statement.c
	migcom.tproj/string.c
	migcom.tproj/type.c
	migcom.tproj/user.c
	migcom.tproj/utils.c
	darling/src/xtracemig.c
	${BISON_MyParser_OUTPUTS}
    ${FLEX_MyScanner_OUTPUTS}
)

add_executable(migcom ${mig_sources})

add_custom_command(
	OUTPUT "/build-mig"
	DEPENDS "/migcom.tproj/mig.sh"
	COMMAND  -E env
		awk
		-v "migcc="
		-v "migcom=/migcom"
		-f "/darling/src/mig.awk"
		"/migcom.tproj/mig.sh"
		> "/build-mig"
	COMMAND  -E env /bin/chmod 0755 "/build-mig"
	)
add_custom_target(migexe DEPENDS "${CMAKE_BINARY_DIR}/build-mig")

add_dependencies(migexe migcom)


