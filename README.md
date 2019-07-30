## About Adon

Adon is a cryptocurrency focused on providing a decentralized mechanism of exchange, and anonymity via untraceable and unlinkable transactions.

You can read more about it at https://adon.network
## About Adon-gui-wallet

This in a multi platform GUI wallet for Adon.

## Building Adon-gui

### 1. Clone wallet sources

```
git clone https://github.com/adon-net/adon-gui-wallet.git
```

### 2. Update git submodules

```
git submodule update --init --recursive
git submodule foreach git pull origin gui
```

### 3. Build

#### On Windows

Dependencies: MSVC 2015 or later, CMake 2.8.6 or later, Boost 1.59 or later and QT 5.1 or later.

You may download them from:

* http://www.microsoft.com/
* http://www.cmake.org/
* http://www.boost.org/
* https://www.qt.io/

To build, change to a directory where this file is located, and run these commands:
```
mkdir build
cd build
cmake -G "Visual Studio 14 Win64" ..
```

If you are building on an older processor without AVX support, add the following options to cmake:
```
-DPORTABLE=1 -DWITH_AVX2=0
```

You may find it helpful to explicitly include Boostand QT paths:
```
cmake.exe -DPORTABLE=1 -Wno-dev -DBOOST_ROOT=C:\boost_1_59_0 -DBOOST_LIBRARYDIR=C:\boost_1_59_0\libs\ -DCMAKE_PREFIX_PATH=C:\Qt\5.10.0\msvc2015_64 -G "Visual Studio 15 Win64" ..
```

And then build from within MSVC.

#### On * nix / OS X

Dependencies: CMake 2.8.6 or later, Boost 1.59 or later and QT 5.1 or later.

You may download them from:

* http://www.cmake.org/
* http://www.boost.org/
* https://www.qt.io/

```
mkdir build && cd build && cmake -DSTATIC=1 .. && make
```

You may find it helpful to explicitly include Boost and QT paths
```
cmake -DSTATIC=1 -DBOOST_ROOT=/boost_1_59_0 -DBOOST_LIBRARYDIR=/boost_1_59_0/libs/ -DCMAKE_PREFIX_PATH=/qt/5.10 ..
```

#### To create a portable build

##### On * nix

```
cp src/adon.desktop build/
cp src/images/adon.png build/
cd build
linuxdeployqt.AppImage adon.desktop -appimage -verbose=2 -always-overwrite -no-translations
```

##### On OS X

```
./macdeployqt Adon.app -dmg
```

##### On Windows

```
windeployqt.exe Adon.exe
```

## License

Adon-gui is licensed under the GNU Lesser General Public License v3.0

