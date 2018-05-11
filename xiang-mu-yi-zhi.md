### VTK编译环境

1、Could NOT find OpenGL (missing: OPENGL_gl_LIBRARY OPENGL_INCLUDE_DIR)

sudo apt-get install libxext-dev libpng-dev libimlib2-dev libglew-dev libxrender-dev libxrandr-dev libglm-dev

2、X11_Xt_LIB could not be found.
sudo apt-get install libxt-dev 

3、sudo make install

cmake -D BUILD_SHARED_LIBS=OFF -D CMAKE_BUILD_TYPE=Release

### ITK编译

cmake -D BUILD_SHARED_LIBS=OFF -D CMAKE_BUILD_TYPE=Release -D Module_ITKReview=ON -D Module_ITKVtkGlue=ON -D Module_MinimalPathExtraction=ON

1、VTK必须在ITK编译前

2、[Modules/IO/TransformBase/src/CMakeFiles/ITKIOTransformBase.dir/itkTransformFactoryBase.cxx.o] Error 4
make[4]: [Modules/IO/TransformBase/src/CMakeFiles/ITKIOTransformBase.dir/all] Error 2
make[3]: [all] Error 2
make[2]: [ITKv4-prefix/src/ITKv4-stamp/ITKv4-build] Error 2
make[1]: [CMakeFiles/ITKv4.dir/all] Error 2
make: [all] Error 2

引起原因：out of memory

### image-analysis

1、CMake Warning at /usr/share/cmake-3.5/Modules/FindBoost.cmake:725 (message):
  Imported targets not available for Boost version
  
  sudo apt-get install libboost-all-dev
  
### bazel安装
1、sudo apt-get install curl
2、sudo apt-get install openjdk-8-jdk
3、echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
4、curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
5、sudo apt-get update && sudo apt-get install bazel