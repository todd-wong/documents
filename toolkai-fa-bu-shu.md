## deepvessel algorithm

### pull code

1. deep-vessel

   * for Seattle Team(recommend)

     repo: git@gitlab.com:youbingy/deep-vessel.git
     username: admin@curacloudcorp.com
     password: Drongeekgen7
    
     Note: you **must** login in https://gitlab.com with by the
           **username and password**, and add ssh key.
    
   * for Beijing Team

     repo: git@internal.curacloudcorp.com:importor/deep-vessel.git
 
 2. itk
    source: http://sourceforge.net/projects/itk/files/itk/4.11/InsightToolkit-4.11.1.tar.gz
    command: wget http://sourceforge.net/projects/itk/files/itk/4.11/InsightToolkit-4.11.1.tar.gz/download && mv download ITK-4.11.1.tar.gz && tar -xzvf ITK-4.11.1.tar.gz

  3. vtk
     source: https://www.vtk.org/files/release/7.1/VTK-7.1.1.tar.gz
     command: wget https://www.vtk.org/files/release/7.1/VTK-7.1.1.tar.gz && tar -xzvf VTK-7.1.1.tar.gz

note: itk and vtk also can download on browsers.

### compile

1. set up environment

    sudo apt-get install libboost-all-dev cmake-curses-gui

2. vtk build statically

    cd {VTK_SRC} && mkdir build && cd build && cmake -D BUILD_SHARED_LIBS=OFF -D CMAKE_BUILD_TYPE=Release .. && sudo make install -j4

3. itk build statically

    cd {ITK_SRC} && mkdir build && cd build && cmake -D BUILD_SHARED_LIBS=OFF -D CMAKE_BUILD_TYPE=Release -D Module_ITKReview=ON -D Module_ITKVtkGlue=ON -D Module_MinimalPathExtraction=ON .. && make install -j4

4. image-analysis build

    cd deep-vessel/image-analysis && mkdir build && cd build && cmake -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_C_FLAGS="-m64" -DCMAKE_CXX_FLAGS="-m64" && make install -j4

    output path: deep-vessel/image-analysis/bin

5. package python scripts, such like:

    pyinstaller --hidden-import=sklearn.neighbors.typedefs --paths="/path/to/deep-vessel" cl_label_feature.py
    
    pyinstaller --hidden-import=sklearn.neighbors.typedefs --paths="/path/to/deep-vessel" --paths='/path/to/deep-vessel/image-analysis/python_bash_prototype/lib' FFR_Test.py
    
    pyinstaller --hidden-import=sklearn.neighbors.typedefs --paths='/path/to/deep-vessel/image-analysis/python_bash_prototype/lib' connectSmallTrees.py
    
    pyinstaller --hidden-import=sklearn.neighbors.typedefs --paths='/path/to/deep-vessel/image-analysis/python_bash_prototype/lib' disconnectSmallTrees.py

## Tool

### pull code

1. Tool

    repo: git@internal.curacloudcorp.com:cc/Tool.git
    branch: dvffr0.2-next

2. sub modules:
  * linux(including itk/qt/quazip/vtk/xcb)
  
    path: Tool/3rdParty/linux:
    repo: git@internal.curacloudcorp.com:cc/Tool-external-linux.git

    Note: remember to rename Tool-external-linux to linux.

  * gyp
    path: Tool/tool/build/gyp
    repo: git@internal.curacloudcorp.com:cc/gyp.git

### compile and run

1. setup compile enviroment

    sudo apt-get install clang libxcb-render-util0 libxcb-image0 libxcb-icccm4  libxcb-shm0 libxcb-keysyms1 libxcb-xinerama0 libxcb-xkb1 libxkbcommon-x11-0
 
2. optionally, cp latest deepvessel algorithm to Tool

   * c++ binary
     cd Tool && cp ${DEEP_VESSEL_SRC}/deep-vessel/image-analysis/bin . -fr 
   * python package
     you need copy them to ./bin
    
2. compile command

    cd Tool && ./tool/package_web.sh
    
3. run command

    cd Tool && ./out/Release_x64/Tool
    
### release

cd Tool && make pack_deb