1、docker run -v /home/curacloud/todd/:/todd --name="todd" -it a33d212fc976 bash

2、docker exec -it 0ea55d2b3dd3 bash

3、docker start 0ea55d2b3dd3

export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64\
                         ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

apt-get install -y python-qt4

ImportError: No module named share.common.utilities


python-qt4 libgl1-mesa-glx

export LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/usr/local/cuda/extras/CUPTI/lib64:/usr/local/lib

export PYTHONPATH=/usr/local/lib/python2.7/dist-packages/vtk

torch-0.3.1-cp27-cp27mu-manylinux1_x86_64.whl


mount -t cifs //10.10.0.139/Share /mnt -o user=ruitaow,password=Wang231147

sudo mount -o username=jung,password=  //10.10.0.139/Share/lijing ~/uploadnas

mac
mount -t smbfs //ruitaow:Wang231147@10.10.0.139/Share ~/NAS

pytest --case=AZBWZ -v Scripts/UnitTest/v0.0.3/test_image_analysis_dv2.py
pytest --case=AZLJZH -v Scripts/UnitTest/v0.0.3/test_image_analysis_dv2.py
pytest --case=AZLXY -v Scripts/UnitTest/v0.0.3/test_image_analysis_dv2.py
pytest --case=AZYZM -v Scripts/UnitTest/v0.0.3/test_image_analysis_dv2.py
pytest --case=AZZDJ -v Scripts/UnitTest/v0.0.3/test_image_analysis_dv2.py

tool编译
1. cd web; npm install
2. apt-get install libxcb-render-util0 libxcb-image0 libxcb-icccm4  libxcb-shm0 libxcb-keysyms1 libxcb-xinerama0 libxcb-xkb1 libxkbcommon-x11-0
3. cd tool/build/;git clone git@internal.curacloudcorp.com:cc/gyp.git
4. cp 3rd_party/linux
4. ./tool/package_web.sh

imageViewer查看mhd文件

itk::CudaResampleImageFilter<itk::Image<short, 3u>, itk::Image<short, 3u>, double, double>::GenerateData

itk::ImageFileReader<itk::Image<short, 3u>, itk::DefaultConvertPixelTraits<short> >::GenerateData

肺结节和脑出血算法路径：/10.10.0.139/share/CuraRad/tmp/CuraRAD

pipreqs生成python依赖库的require.txt