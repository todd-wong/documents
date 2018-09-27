1、dicom_curator.py dicom_folder output_dir side series1.instance1 series2.instance2

从DICOM image中读取对应series1.instance1和series2.instance2并生成对应的png图片，实际输出在Series目录

2、auto_frame_selection.py series



origin_dir: dicom路径
processing_dir: 数据结果路径
patient_id: 病人id


pipeline
view_selector.py dicom_dir
run_3drecon.sh patientid_param.txt -2d
run_3drecon.sh patientid_param.txt -3d
python ./main_dvqfr.py --file ~/QFFRData/BJHXY/LCX_param.txt --setup
python ./main_dvqfr.py --file ~/QFFRData/BJHXY/LCX_param.txt --fcount
//python ./main_dvqfr.py --file ~/QFFRData/BJHXY/LCX_param.txt --cfd
python ./main_dvqfr.py --file ~/QFFRData/BJHXY/LCX_param.txt --dl
python ./main_dvqfr.py --file ~/QFFRData/BJHXY/LCX_param.txt --em