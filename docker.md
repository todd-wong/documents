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


username: admin@curacloudcorp.com pw: Drongeekgen7

curl 10.199.10.10:80/analysis -d IaSVT5%2BWWcp0bmIBBlzcnA%3D%3D%7CUI5o0SgOUqDQL9doXuxLSKohg81BWZQh2uXCdzsUYU9pty%2BEQRTOGmcpejshTy8ajR1C%2BfvSrJTkZY9Jyq5jlj%2Fo2lmA%2Batsou4AJl7VNrErO8%2Bb6Us6kw8hv8HMK%2Bx1hx%2B8HOetQoyZoKlK54659vlJsBgpaPylwyP%2Bdsr6o3w93b4YrGEH1cRjdibakeXlZfrce3a%2FAzPr4c%2FZlTwOtXFxUHEHyxw9cIopLgHE2UqEMZisPj9aK5gPJScvgk7cbFY2dUvSbfBO01hgQgdgJvsIb795ztHtgbLgxWaJ3e5fGbRbe6GTTw3JrsRVzPWvR9qv7C7ISGkqv3GGIm5wOA%3D%3D

curarad-ich discussion(2018-9-28):
1、 mrn, accession 确认是否required
2、 确认dicom接收是否设置timeout
3、 软件实现的细节补充
4、 comments的答复

reply: ich-server and pacs-server are wrapped in to each docker, but AI algorithm scripts are placed in the directory. 

transferring encrypted cases meta info with aes-256-cbc(base64)

case info:
 {
   "uid":"1.2.123.117531.20.35651.53481.20170712.140521.3570254",
     "mrn":"A114789",
     "accession":"A12155781",
     "type": "ICH",
     "imageCount":100,
     "addition": {
       "procedureName": "procedure name",
       "modality": "CT",
       "examReason": "",
       "contrast": "",
       "gender": "M",
       "age": 40,
       "history": "",
     }
 }
ich-server just require uid(study instance uid) and imageCount, but clario need mrn and accession for their matching info
当前ich-server只需要uid(study instance uid)和imageCount字段，uid用于分辨具体case，而imageCount用于判断传输数据是否完成。
mrn和accession字段是clario方需要携带的信息，用于匹配
type是基于有可能不止ich一个项目的考虑
addition可去

mysql用于记录分析请求
多个请求排队执行，始终只有一个在运行算法
若一个请求数据未完成，会始终停留在waiting状态，没做任何超时处理，通过mysql可查询waiting的task。数据库中的item未添加时间戳

waiting->pending->run->finished

logs

tensorflow extension

当clario端send request，服务端会记录请求，并标注为Waiting状态，插入等待队列，一旦接收完dicom数据，后台会以间隔2s的Timer(程序初始化启动)，循环检测传输dicom数据是否完成，并将记录标为Pending状态。如果暂时未有case在运行算法，则该case立即执行，并将状态标为Running状态，完成后，发送结果给clario客户端；如果有其他case在运行，则等待下一轮check。

const json = {
    uid: req.uid,
    mrn: req.mrn,
    accession: req.accession,
    status: req.status,
    result: req.result || '',
    prioritization: req.prioritization != null ? req.prioritization : -1,
  };


study instance uid
imageCount

确认部署

queue的长度

测试（单元测试。。。）

framework

nodejs test framework
mock + SuperTest + Istanbul

test src path:
/src/test/router.js

1. Analysis API(/analysis)

Analysis API is to receive information on the incoming exam. It accepts the following fields: 

method: POST

params:

uid (required)  

The unique identifier to link exam information and DICOM data. 

mrn (required)  

The MRN of the patient. 

accession (required)  

The accession number of the patient. 

imageCount (required)  

Total DICOM data files count. AI analysis will not be performed until all image files are transferred. 

addition (required)  

The information from EMR/PACS structured as JSON object. 

Sample JSON payload: 

{ 

"mrn":"A114789", 

"accession":"A12155781", 

"type": "ICH", 

"imageCount":100, 

"addition": { 

"procedureName": "procedure name", 

"modality": "CT", 

"examReason": "", 

"contrast": "", 

"gender": "M or F", 

"age": 40, 

"history": "", 

} 

} 

output:
head:
status code: 200

body:
{status: 'ok'}

error output:
head:
status code: 400
body:
error messages(json)

2. Feedback API(/feedback)

Feedback result from client

method: POST

params:

Feedback API accepts the following fields: 

uid (required)  

The unique identifier to link exam information and DICOM data. 

mrn (required) 

The MRN of the patient. 

accession (required) 

The accession number of the patient.  

report (required) 

The final radiology report which is written by radiologists. 

Sample JSON payload: 

{ 

"mrn":"A114789", 

"accession":"A12155781", 

"report": "", 

} 

output:
head:
status code: 200
body:
{status: 'ok'}

error output:
head:
status: 400
body:
error messages(json)

3. Algorithm API(deepICH.py)

Host predict_server
  Hostname 10.199.10.9
  User ubuntu
  Port 22
  
  
  
  
  https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html
python /data/datasets/ICH_510k_pipeline/scripts/dicom_to_nifti.py /data/datasets/ICH_510k_pipeline/dicom/CH_POS_1 /data/datasets/ICH_510k_pipeline/tmp_dicom_series /data/datasets/ICH_510k_pipeline/tmp_nifti_files  这里面CH_POS_1是放原始dicom数据的，tmp_dicom_series和tmp_nifiti_files这两个路径是用来盛放中间输出的。






https://linuxconfig.org/nvidia-geforce-driver-installation-on-centos-7-linux-64-bit

login as root

nvidia driver installation:

yum update
yum install kernel-devel-$(uname -r) gcc
echo 'blacklist nouveau' >> /etc/modprobe.d/blacklist.conf
dracut /boot/initramfs-$(uname -r).img $(uname -r) --force
reboot
./NVIDIA-Linux-x86_64-384.111.run
nvidia-smi

pip installation:
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python get-pip.py

docker-ce installation:

sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce

sudo systemctl start docker

sudo docker run hello-world

docker-compose installation:

sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version

docker-compose installation:

distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | \
  sudo tee /etc/yum.repos.d/nvidia-docker.repo
  
sudo yum install nvidia-docker2
sudo pkill -SIGHUP dockerd

