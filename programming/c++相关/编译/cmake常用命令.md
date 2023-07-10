<https://blog.csdn.net/u011436427/article/details/125838619>\
<https://www.bilibili.com/video/BV16P4y1g7MH/?spm_id_from=333.788&vd_source=ac44a38168194f76fada7686c12a4a37>
### 编译运行
```shell
#古代 camke

#项目根目录下执行
mkdir -p build
cd build
#根据camkelists文件生成makefile
camke .. -DCMAKE_VUILD_TYPE=Release
make -j4 #并行编译生成可执行文件
make install #安装
#找到可执行文件运行
.可执行文件.exe/out

#现代cmake
camke -B build -DCMAKE_VUILD_TYPE=Release

cmake --build build --parallel 4

cmake --build build --target install
#找到可执行文件运行
.可执行文件.exe/out
```