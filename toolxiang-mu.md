## Tool编译
1. cd web; npm install
2. apt-get install libxcb-render-util0 libxcb-image0 libxcb-icccm4  libxcb-shm0 libxcb-keysyms1 libxcb-xinerama0 libxcb-xkb1 libxkbcommon-x11-0
3. cd tool/build/;git clone git@internal.curacloudcorp.com:cc/gyp.git
4. cp 3rd_party/linux
4. ./tool/package_web.sh

## Tool调试
* 在tool_linux.gyp中添加cflag -g
* 在out/Release_x64/deepvesselffr中修改：
  $APP_DIR/Loader $1 $2 为 gdb $APP_DIR/Loader $1 $2
* 若在gdb调试中出现，不能打印std::string格式字符串，则在Makefile中用g++编译，而不是clang
* 方便打印QString字符串，添加～/.gdbinit，内容如下:

```

define printqstring 
    printf "(QString)0x%x (length=%i): \"",&$arg0,$arg0.d->size
    set $i=0
    while $i < $arg0.d->size
        set $c=$arg0.d->data()[$i++]
        if $c < 32 || $c > 127
                printf "\\u0x%04x", $c
        else
                printf "%c", (char)$c
        end
    end
    printf "\"\n"
end

```