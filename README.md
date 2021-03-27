unreal-engine with pixelstreaming patches for linux
===================================================

Based on:

* https://aur.archlinux.org/packages/unreal-engine/

Builds engine with pixelstreaming patches for linux from:

* https://github.com/ImmortalEmperor/UnrealEngine
* https://github.com/ImmortalEmperor/UnrealEngine/tree/4.25-pixelstreaming

Build with:

        sudo ln -s /opt/cuda /usr/local/
        makepkg [ -C ]
