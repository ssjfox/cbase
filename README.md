
Build Solution.

* multi platform : windows_x86_64 , linux_x86_64 , linux_aarch64
* build language : c/c++
* build script : bash + cmake
* GUI build : only QT ( uic , moc with clean build )
* Suitable editor : vscode
* option : ninja , available cross compile 

```
$ bash ./.script/_b [clean] [build] [run] [debug]

$ bash ./.script/_b clean
$ bash ./.script/_b clean build
$ bash ./.script/_b clean build run
$ bash ./.script/_b clean build run debug
$ bash ./.script/_b clean build debug
$ bash ./.script/_b clean debug
$ bash ./.script/_b run
$ bash ./.script/_b run debug
$ bash ./.script/_b build run
$ bash ./.script/_b build
$ ...
```

# for Windows 

* install git-scm
* git-bash with vs2019

