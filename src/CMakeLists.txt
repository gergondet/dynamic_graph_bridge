PYTHON_INSTALL("dynamic_graph/ros" "__init__.py" "${PYTHON_SITELIB}" "${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_PYTHON_DESTINATION}/../")
PYTHON_INSTALL("dynamic_graph/ros" "ros.py" "${PYTHON_SITELIB}" "${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_PYTHON_DESTINATION}/../")
FILE(COPY "dynamic_graph/ros/__init__.py" "dynamic_graph/ros/ros.py"
     DESTINATION "${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_PYTHON_DESTINATION}/../dynamic_graph/ros/")
