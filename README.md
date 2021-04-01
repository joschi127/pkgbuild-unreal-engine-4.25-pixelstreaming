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

Install with:

        sudo pacman -U unreal-engine-4.25-pixelstreaming-4.25-4-x86_64.pkg.tar.zst

Fix permissions: (to avoid errors when building packages, etc.)

        sudo chown -R $(whoami):$(whoami) /opt/unreal-engine

Fix / add desktop shortcut:

        cp -vf /opt/unreal-engine/Engine/Source/Programs/UnrealVersionSelector/Private/Linux/Resources/*.desktop ~/.local/share/applications/
