set (CLICKHOUSE_ODBC_BRIDGE_SOURCES
    ColumnInfoHandler.cpp
    getIdentifierQuote.cpp
    HandlerFactory.cpp
    IdentifierQuoteHandler.cpp
    MainHandler.cpp
    ODBCBlockInputStream.cpp
    ODBCBridge.cpp
    PingHandler.cpp
    validateODBCConnectionString.cpp
)

set (CLICKHOUSE_ODBC_BRIDGE_LINK
    PRIVATE
        clickhouse_parsers
        daemon
        dbms
        Poco::Data
    PUBLIC
        Poco::Data::ODBC
)

clickhouse_program_add_library(odbc-bridge)

if (OS_LINUX)
    # clickhouse-odbc-bridge is always a separate binary.
    # Reason: it must not export symbols from SSL, mariadb-client, etc. to not break ABI compatibility with ODBC drivers.
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--no-export-dynamic")
endif ()

add_executable(clickhouse-odbc-bridge odbc-bridge.cpp)
set_target_properties(clickhouse-odbc-bridge PROPERTIES RUNTIME_OUTPUT_DIRECTORY ..)

clickhouse_program_link_split_binary(odbc-bridge)

if (USE_GDB_ADD_INDEX)
    add_custom_command(TARGET clickhouse-odbc-bridge POST_BUILD COMMAND ${GDB_ADD_INDEX_EXE} ../clickhouse-odbc-bridge COMMENT "Adding .gdb-index to clickhouse-odbc-bridge" VERBATIM)
endif()

install(TARGETS clickhouse-odbc-bridge RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)

if(ENABLE_TESTS)
    add_subdirectory(tests)
endif()
