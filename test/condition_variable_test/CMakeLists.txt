
# Find the dependencies

find_package(GTest REQUIRED)

# Configure test target

add_executable (condition_variable_test main.cc condition_variable_test.cc)
target_link_libraries (condition_variable_test wh::utils GTest::GTest GTest::Main)

add_test (
    NAME condition_variable_test
    COMMAND $<TARGET_FILE:condition_variable_test>
)

if(MSVC)
    set_target_properties(condition_variable_test PROPERTIES
        FOLDER Tests
    ) 
endif()