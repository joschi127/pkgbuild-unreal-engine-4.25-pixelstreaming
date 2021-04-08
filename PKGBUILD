# Maintainer: Dylan Ferris <dylan@psilly.com>
# Maintainer: Michael Lojkovic <mikelojkovic@gmail.com>
# Maintainer: Shatur95 <genaloner@gmail.com>

# The source is about 200 MiB, with an extra ~11 GiB of dependencies downloaded in Setup.sh, and may take several hours to compile.
# Build with:
#   sudo ln -s /opt/cuda /usr/local/
#   makepkg [ -C ]
pkgname=unreal-engine-4.25-pixelstreaming
pkgver=4.25
pkgrel=4
#branchname=4.25-pixelstreaming
branchname=vulkan-memory-leak-fix
pkgdesc='A 3D game engine by Epic Games which can be used non-commercially for free.'
arch=(x86_64)
url=https://www.unrealengine.com/
makedepends=(mono dos2unix git openssh msbuild)
depends=(icu sdl2 python lld xdg-user-dirs cuda cuda-tools cudnn opencl-nvidia)
optdepends=('qt5-base: qmake build system for projects'
            'cmake: build system for projects'
            'qtcreator: IDE for projects'
            'codelite: IDE for projects'
            'kdevelop: IDE for projects'
            'clion: IDE for projects')
provides=("unreal-engine=$pkgver-$pkgrel")
conflicts=(unreal-engine)

license=(custom:UnrealEngine)
source=('com.unrealengine.UE4Editor.desktop'
        'use-arch-mono.patch'
        'clang_11.patch'
        'PackageWithSystemCompiler.patch'
        'ccache.patch'
        'compile_in_editor.patch'
        '430667-13636743.patch')
sha256sums=('15e9f9d8dc8bd8513f6a5eca990e2aab21fd38724ad57d213b06a6610a951d58'
            'e891f07bf7294cd5fde8eb6de92e6d47ed004847ea8afd7c944e9b9b2bacaff4'
            '8042bed3405298b5a4357068dd6b22a5a8a0f19def64b4f61ed0362fb46cb00d'
            '9e403b939a0601c6271da17af9497742cacd74e3cde41562c9f288dfbdcbdbfe'
            'a0a0d3f065e27f4d31e21e5f9d15cb4d8f59c50245a45469878fc1fe8bdc78e6'
            'e4153fce86147dec7bf7dade1fc9aeac3e8f9a68d75ae92f60a6f070ff6378a4'
            'SKIP')
options=(!strip staticlibs) # Package is 3 Gib smaller with "strip" but it takes a long time and generates many warnings

# Set options to anything that is not null to enable them.
_system_compiler=         # for the system compiler you'll need to set LINUX_MULTIARCH_ROOT
                           # as an environment to /usr/sbin compile projects after building.
_ccache_support=       # Patches for ccache. More optimizations might be needed.

prepare() {
  # Check access to the repository
  if ! git ls-remote git@github.com:EpicGames/UnrealEngine &>-
  then
    error 'You must register at unrealengine.com and link your github account to access this private repo. See the wiki for more info: https://wiki.archlinux.org/index.php/Unreal_Engine_4'
    exit 1
  fi

  # Download Unreal Engine source or update if the folder exists
  if [ ! -d $pkgname ]
  then
    git clone --depth=1 --branch=$branchname git@github.com:ImmortalEmperor/UnrealEngine $pkgname
    cd $pkgname
  else
    cd $pkgname
    #rm -f .git/index.lock
    #git fetch --depth=1 origin tag $branchname
    git remote remove origin
    git remote add origin git@github.com:ImmortalEmperor/UnrealEngine
    git remote update
    git reset --hard origin/$branchname
    git merge origin/4.25-pixelstreaming-dev || true
    git checkout --theirs Engine/Plugins/Media/PixelStreaming/Source/PixelStreaming/Private/Codecs/VideoEncoder.cpp
    git add Engine/Plugins/Media/PixelStreaming/Source/PixelStreaming/Private/Codecs/VideoEncoder.cpp
  fi

  #patch Engine/Build/BatchFiles/Linux/SetupMono.sh $srcdir/use-arch-mono.patch # Use system mono
  generateProjectArgs="-makefile"
  if [ -n "$_system_compiler" ]
  then
    patch -p1 -i "$srcdir/clang_11.patch"
    patch -p9 -i "$srcdir/PackageWithSystemCompiler.patch"
    patch -p8 -i "$srcdir/compile_in_editor.patch"
    export LINUX_MULTIARCH_ROOT="/usr/sbin"
    generateProjectArgs+=" -ForceUseSystemCompiler"
  fi
  if [ -n "$_ccache_support" ]
  then
    patch -p1 -i "$srcdir/ccache.patch"
  fi

  # Patch for crash: Malloc Size=65538 LargeMemoryPoolOffset=65554 / VulkanUtil.cpp] [Line: 770] / Failed with Validation error. Try running with r.Vulkan.EnableValidation=1 to get information from the driver
  #patch --strip=4 -i "$srcdir/430667-13636743.patch"

  # Qt Creator source code access
  if [ ! -d Engine/Plugins/Developer/QtCreatorSourceCodeAccess ]
  then
    git -C Engine/Plugins/Developer clone --depth=1 git@github.com:fire/QtCreatorSourceCodeAccess
  fi

  export TERM=xterm
  ./Setup.sh
  ./GenerateProjectFiles.sh $generateProjectArgs
}

build() {
  cd $pkgname

  # Build all targets from the "all" rule separately, because building multiple targets in parallel results in an error (but building one target with multiple threads is possible)
  ARGS=""
  if [ -n "$_system_compiler" ]
  then
    ARGS="ARGS=-ForceUseSystemCompiler"
  fi
  make $ARGS CrashReportClient-Linux-Shipping
  make $ARGS CrashReportClientEditor-Linux-Shipping
  make $ARGS ShaderCompileWorker
  make $ARGS UnrealLightmass
  make $ARGS UnrealFrontend
  make $ARGS UE4Editor
  make $ARGS UnrealInsights
}

package() {
  # Install dir
  #dir="opt/$pkgname"
  dir="opt/unreal-engine"

  # Desktop entry
  if [ "$dir" != "opt/$pkgname" ] # Set new path if dir changed
  then
    sed -i "5c\Path=/$dir/Engine/Binaries/Linux/" com.unrealengine.UE4Editor.desktop
    sed -i "6c\Exec=/$dir/Engine/Binaries/Linux/UE4Editor %F" com.unrealengine.UE4Editor.desktop
  fi
  install -Dm777 com.unrealengine.UE4Editor.desktop $pkgdir/usr/share/applications/com.unrealengine.UE4Editor.desktop

  cd $pkgname

  # Icon for Desktop entry
  install -Dm777 Engine/Source/Programs/UnrealVS/Resources/Preview.png $pkgdir/usr/share/pixmaps/ue4editor.png

  # License
  install -Dm777 LICENSE.md $pkgdir/usr/share/licenses/UnrealEngine/LICENSE.md

  # Engine
  install -dm777 "$pkgdir/$dir/Engine"
  mv Engine/Binaries "$pkgdir/$dir/Engine/Binaries"
  mv Engine/Build "$pkgdir/$dir/Engine/Build"
  mv Engine/Config "$pkgdir/$dir/Engine/Config"
  mv Engine/Content "$pkgdir/$dir/Engine/Content"
  mv Engine/Documentation "$pkgdir/$dir/Engine/Documentation"
  mv Engine/Extras "$pkgdir/$dir/Engine/Extras"
  mv Engine/Plugins "$pkgdir/$dir/Engine/Plugins"
  mv Engine/Programs "$pkgdir/$dir/Engine/Programs"
  mv Engine/Shaders "$pkgdir/$dir/Engine/Shaders"
  mv Engine/Source "$pkgdir/$dir/Engine/Source"

  # Required folders
  # install -d "$pkgdir/$dir/Engine/DerivedDataCache"

  # Content
  mv FeaturePacks "$pkgdir/$dir/FeaturePacks"
  mv Samples "$pkgdir/$dir/Samples"
  mv Templates "$pkgdir/$dir/Templates"

  # Build scripts, used by some plugins (CLion)
  install -Dm777 GenerateProjectFiles.sh "$pkgdir/$dir/GenerateProjectFiles.sh"
  install -Dm777 Setup.sh "$pkgdir/$dir/Setup.sh"
  install -Dm777 .ue4dependencies "$pkgdir/$dir/.ue4dependencies"
}
