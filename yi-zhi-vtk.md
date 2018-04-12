### Emscripten编译vtk Project
1、git clone vtk source。
2、ccmake target dir，设置BUILD_TESTING=OFF，CMAKE_BUILD_TYPE=Release，CMAKE_INSTALL_PREFIX=/target/path/to/vtk/build。
3、emmake make
4、make install

最终在/target/path/to/vtk/build目录生成vtk的头文件和libs。

注：中间编译会出现编译文件缺失，可通过正常编译模式复制。

### Binding vtk接口
vtk_wrap.cc:
``` cplusplus

#include <iostream>                                                                                                                                   
#include <memory>
 
#include <emscripten/bind.h>
 
#include <vtkAlgorithmOutput.h>
#include <vtkAlgorithm.h>
#include <vtkSmartPointer.h>
#include <vtkNIFTIImageReader.h>
#include <vtkMarchingCubes.h>
#include <vtkGeometryFilter.h>
#include <vtkPLYWriter.h>

namespace emscripten {
    template<typename T>
    struct smart_ptr_trait<vtkSmartPointer<T>> {
        typedef vtkSmartPointer<T> pointer_type;
        typedef T element_type;
 
        static sharing_policy get_sharing_policy() {
            return sharing_policy::NONE;
        }
 
        static T* get(const vtkSmartPointer<T>& p) {
            return p.Get();
        }
 
        static vtkSmartPointer<T> share(const vtkSmartPointer<T>& r, T* ptr) {
            return vtkSmartPointer<T>(ptr);
        }
 
        static pointer_type* construct_null() {
            return new pointer_type;
        }
    };
 
#define VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(class_name) \
namespace internal {                                       \
template<>                                                 \
void raw_destructor<class_name>(class_name* ptr) {         \
    ptr->UnRegister(nullptr);                              \
}                                                          \
}

VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkObjectBase)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkObject)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkDataObject)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkDataSet)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkPointSet)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkPolyData)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkAlgorithmOutput)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkAlgorithm)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkImageAlgorithm)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkImageReader2)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkNIFTIImageReader)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkPolyDataAlgorithm)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkMarchingCubes)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkGeometryFilter)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkWriter)
VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE(vtkPLYWriter)
 
#undef VTK_RAW_DESTRUCTOR_TEMPLATE_INITIALIZE
} //namespace emscripten

EMSCRIPTEN_BINDINGS(vtkSmartPointer) {
using namespace emscripten;
 
class_<vtkObjectBase>("vtkObjectBase")
   // .smart_ptr_constructor("vtkSmartPointer<vtkObjectBase>", &vtkSmartPointer<vtkObjectBase>::New)  
    .function("GetClassName", optional_override([](vtkObjectBase& this_) {
            return std::string(this_.GetClassName());
        }))
    ;
 
class_<vtkObject, base<vtkObjectBase>>("vtkObject")
    .function("PrintSelf", optional_override([](vtkObject& this_) {
            this_.PrintSelf(std::cout, *vtkIndent::New());
        }))
    ;
 
class_<vtkDataObject, base<vtkObject>>("vtkDataObject")
    .class_function("New", &vtkDataObject::New, allow_raw_pointer<ret_val>())
    ;
 
class_<vtkDataSet, base<vtkDataObject>>("vtkDataSet")
    .function("GetNumberOfCells", &vtkDataSet::GetNumberOfCells, pure_virtual())
    ;
 
class_<vtkPointSet, base<vtkDataSet>>("vtkPointSet")
    ;
 
class_<vtkPolyData, base<vtkPointSet>>("vtkPolyData")
    .class_function("New", &vtkPolyData::New, allow_raw_pointer<ret_val>())
 //   .function("GetNumberOfCells", &vtkPolyData::GetNumberOfCells)
    ;

class_<vtkAlgorithmOutput, base<vtkObject>>("vtkAlgorithmOutput")
    .class_function("New", &vtkAlgorithmOutput::New, allow_raw_pointer<ret_val>())
    ;
 
class_<vtkAlgorithm, base<vtkObject>>("vtkAlgorithm")
    .function("GetNumberOfInputPorts", &vtkAlgorithm::GetNumberOfInputPorts)
    .function("GetNumberOfOutputPorts", &vtkAlgorithm::GetNumberOfOutputPorts)
//    .function("Register", &vtkAlgorithm::Register, allow_raw_pointers())
//    .function("UnRegister", &vtkAlgorithm::UnRegister, allow_raw_pointers())
    .function("Update", select_overload<void()>(&vtkAlgorithm::Update))
    .function("Update_i", select_overload<void(int)>(&vtkAlgorithm::Update))
    .function("GetOutputPort", select_overload<vtkAlgorithmOutput*()>(&vtkAlgorithm::GetOutputPort), allow_raw_pointers())
    .function("SetInputConnection", select_overload<void(vtkAlgorithmOutput*)>(&vtkAlgorithm::SetInputConnection), allow_raw_pointers())
    ;
 
class_<vtkImageAlgorithm, base<vtkAlgorithm>>("vtkImageAlgorithm")
//    .function("GetOutput", select_overload<vtkImageData* ()>(&vtkImageAlgorithm::GetOutput), allow_raw_pointers())
//    .function("GetOutput_i", select_overload<vtkImageData* (int)>(&vtkImageAlgorithm::GetOutput), allow_raw_pointers())
//    .function("SetOutput", &vtkImageAlgorithm::SetOutput, allow_raw_pointers())
//    .function("SetInputData_d", select_overload<void(vtkDataObject*)>(&vtkImageAlgorithm::SetInputData), allow_raw_pointers())
//    .function("SetInputData_id", select_overload<void(int, vtkDataObject*)>(&vtkImageAlgorithm::SetInputData), allow_raw_pointers())
    ;

class_<vtkImageReader2, base<vtkImageAlgorithm>>("vtkImageReader2")
    .smart_ptr_constructor("vtkSmartPointer<vtkImageReader2>", &vtkSmartPointer<vtkImageReader2>::New)
    .function("SetFileName", optional_override([](vtkImageReader2& this_, const std::string& file_name) {
             this_.SetFileName(file_name.c_str());
         }))
//    .function("SetMemoryBuffer", &vtkImageReader2::SetMemoryBuffer, allow_raw_pointers())
    .function("GetMemoryBuffer", &vtkImageReader2::GetMemoryBuffer, allow_raw_pointers())
    ;
 
class_<vtkNIFTIImageReader, base<vtkImageReader2>>("vtkNIFTIImageReader")
    .smart_ptr_constructor("vtkSmartPointer<vtkNIFTIImageReader>", &vtkSmartPointer<vtkNIFTIImageReader>::New)
    ;
 
class_<vtkNIFTIImageReaderWrapper, base<vtkNIFTIImageReader>>("vtkNIFTIImageReaderWrapper")
    //.smart_ptr_constructor("vtkSmartPointer<vtkNIFTIImageReaderWrapper>", &vtkSmartPointer<vtkNIFTIImageReaderWrapper>::New)
    .constructor()
    ;

class_<vtkPolyDataAlgorithm, base<vtkAlgorithm>>("vtkPolyDataAlgorithm")
    .smart_ptr_constructor("vtkSmartPointer<vtkPolyDataAlgorithm>", &vtkSmartPointer<vtkPolyDataAlgorithm>::New)
    .function("GetOutput", select_overload<vtkPolyData*()>(&vtkPolyDataAlgorithm::GetOutput), allow_raw_pointers())
    ;
 
class_<vtkMarchingCubes, base<vtkPolyDataAlgorithm>>("vtkMarchingCubes")
    .smart_ptr_constructor("vtkSmartPointer<vtkMarchingCubes>", &vtkSmartPointer<vtkMarchingCubes>::New)
    .function("SetValue", &vtkMarchingCubes::SetValue)
    ;
 
class_<vtkGeometryFilter, base<vtkPolyDataAlgorithm>>("vtkGeometryFilter")
    .smart_ptr_constructor("vtkSmartPointer<vtkGeometryFilter>", &vtkSmartPointer<vtkGeometryFilter>::New)
    ;
 
class_<vtkWriter, base<vtkAlgorithm>>("vtkWriter")
    .function("Write", &vtkWriter::Write)
    ;
 
class_<vtkPLYWriter, base<vtkWriter>>("vtkPLYWriter")
    .smart_ptr_constructor("vtkSmartPointer<vtkPLYWriter>", vtkSmartPointer<vtkPLYWriter>::New)
    .function("SetFileTypeToASCII", &vtkPLYWriter::SetFileTypeToASCII)
    .function("SetFileName", optional_override([](vtkPLYWriter& this_, const std::string& file_name) {
            this_.SetFileName(file_name.c_str());
        }))
    ;
}

```

### 编译
Makefile:
``` Makefile

CC = em++
CFLAGS = -std=c++11 --bind -Os -s ASSERTIONS=1 -s WASM=1 -s ALLOW_MEMORY_GROWTH=1 --preload-file lungSeg.nii
INC = -I/Users/todd.wong/vtk/include/vtk-9.0
LIB_PATH = -L/Users/todd.wong/vtk/lib
VTK_LIBS = -lvtksys-9.0 -lvtkCommonCore-9.0 -lvtkCommonMath-9.0 -lvtkCommonSystem-9.0 -lvtkCommonMisc-9.0 -lvtkCommonDataModel-9.0 -lvtkCommonExecutionModel-9.0 -lvtkIOCore-9.0 -lvtkFiltersCore-9.0 -lvtkFiltersGeneral-9.0 -lvtkFiltersGeometry-9.0 -lvtkIOXML-9.0  -lvtkImagingCore-9.0 -lvtkImagingStatistics-9.0 -lvtkmetaio-9.0 -lvtkIOImage-9.0 -lvtkIOPLY-9.0 -lvtkzlib-9.0
SRCS = vtk_wrap.cc

index.html : $(SRCS)
  $(CC) $^ $(LIB_PATH) $(VTK_LIBS) $(INC) $(CFLAGS) -o $@
 
.PHONY : all
default all : index.html
 
.PHONY : clean
clean :
  rm index.html index.js
 
.PHONY : run
run :
  emrun --no_emrun_detect --no_browser --port 8888 index.html

```
执行make，最终生成index.html

### 运行
1、打开本地web服务，运行make run。
2、在浏览器中输入localhost:8888，打开Devtools，执行以下命令
``` javascript

var reader = new Module.vtkNIFTIImageReader();
var discreteCubes = new Module.vtkMarchingCubes();
var geometry = new Module.vtkGeometryFilter();
var writer = new Module.vtkPLYWriter();
 
reader.SetFileName("lungSeg.nii");
reader.Update();
reader.PrintSelf();
 
discreteCubes.SetInputConnection(reader.GetOutputPort());
discreteCubes.SetValue(1, 1);
 
geometry.SetInputConnection(discreteCubes.GetOutputPort());
console.time("Update")
geometry.Update();
console.timeEnd("Update");
geometry.GetOutput().GetNumberOfCells();                                                                                                              
 
writer.SetInputConnection(geometry.GetOutputPort());
writer.SetFileTypeToASCII();
writer.SetFileName("1.ply");
writer.Write();

```

### 结果
Chrome: Update: 12357.110107421875ms
Firefox: Update: 11586ms
native: Update: 5.~s