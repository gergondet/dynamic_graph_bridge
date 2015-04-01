PYTHON_INSTALL("dynamic_graph/ros" "__init__.py" "${PYTHON_SITELIB}")
PYTHON_INSTALL("dynamic_graph/ros" "ros.py" "${PYTHON_SITELIB}")
FILE(COPY "dynamic_graph/ros/__init__.py" "dynamic_graph/ros/ros.py"
     DESTINATION "${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION}/dynamic_graph/ros/")
