# Ultimaker Cura for ARM64
Ultimaker Cura for ARM64 based Single Board Computers

Purpose of this repository is to share Ultimaker Cura AppImage for ARM64 devices and instructions to build both Ultimaker Cura and AppImage.


## Building Cura

These instruction are made with Orange Pi 5 Pro 16GB board running Ubuntu Jammy (Orange Pi OS version). Here are the steps that I did, some might be unnecessary but these should get build complete and Cura running.

In this build Cura git clone is made in ~/Development so build dir is ~/Development/Cura and Qt6 is installed in /opt/Qt (Online installer needs to run with sudo)

#### Python and pip version used in build
- Python 3.10.12
- Pip 24.1.1 (default was 22.x, need to upgrade after first failed build)

#### Install Qt6 binaries with online installer. I used Qt 6.7.0 in this install.
https://www.qt.io/download-qt-installer-oss

At least under additional libraries all should be checked.

#### Install and configure Conan
```
pip install conan==1.64
conan config install https://github.com/ultimaker/conan-config.git
conan profile new default --detect --force
```

In .conan/conan.conf change level-option in [log]-section, this helps to see when build will wait for license input. If not stopped, it stays there waiting forever and consumes all RAM and in the end, your machines is not responsive anymore.
```
[log]
run_to_output = True        # environment CONAN_LOG_RUN_TO_OUTPUT
run_to_file = False         # environment CONAN_LOG_RUN_TO_FILE
level = 10                # environment CONAN_LOGGING_LEVEL, default is warn
print_run_commands = True   # environment CONAN_PRINT_RUN_COMMANDS
```

#### Clone Cura
```
git clone https://github.com/Ultimaker/Cura.git
cd Cura
```

#### Add Qt install path to PATH and set makeflags to compile with all processors to speed up build (optional)
```
export PATH=/opt/Qt/6.7.0/gcc_arm64/bin:/opt/Qt/6.7.0/gcc_arm64/lib:$PATH
export MAKEFLAGS="-j$(nproc)"
```

#### Start build and initialize the Virtual Python Development Environment
If you prefer, you can change to specific branch, e.g. 5.7
```
git checkout 5.7
```
and/or build specific version with
```
conan install . cura/5.7.2@ultimaker/stable --build=missing --update -g VirtualPythonEnv
```
or build latest from master branch with
```
conan install . --build=missing --update -o cura:devtools=True -g VirtualPythonEnv
```

Once build hangs on PyQt6 license checking, stop building with 'Ctrl+C'

#### Upgrade Pip
```
~/Development/Cura/venv/bin/python -m pip install --upgrade pip
```

#### Install PyQt6

First, remove/comment out all PyQt6 modules except PyQt6 itself and PyQt6-sip from Cura and Uranium requirements.txt files
```
~/Development/Cura/requirements.txt
find ~/.conan/ -name requirements.txt # I have this at path ~/.conan/data/uranium/5.7.0/_/_/package/e541e632dd3ca870d37dab822cdf6eaa3df15dca/pip_requirements/requirements.txt
```

#### Change PyQt6==6.6.0 to PyQt6==6.7.0 and change in both files hash to
```
    --hash=sha256:3d31b2c59dc378ee26e16586d9469842483588142fc377280aad22aaf2fa6235
```

#### Remove possible Qt files from venv
```
sudo rm -r venv/lib/python3.10/site-packages/PyQt6*
```

#### Purge pip cache
```
~/Development/Cura/venv/bin/pip cache purge
```

#### Install PyQt6==6.7.0
```
~/Development/Cura/venv/bin/pip install PyQt6==6.7.0 --force-reinstall --config-settings --confirm-license= --verbose
```

#### Restart conan build
```
conan install . --build=missing --update -o cura:devtools=True -g VirtualPythonEnv
```

#### Activate the Virtual Python Environment
```
source venv/bin/activate
```

#### Run Cura
```
python cura_app.py
```

## Building Cura AppImage after building Cura using above instructions

Tested with:
- Orange Pi 5 Pro
- Orange Pi Ubuntu Jammy

#### Build Ultimaker-Cura folder with all necessary files for standalone use
```
cd ~/Development/Cura
pyinstaller venv/conan/UltiMaker-Cura.spec
```
If you get error message about Qt5 you need to exclude it in 'Analysis' section in the end of UltiMaker-Cura.spec-file
```
excludes=['PyQt5'],
```

PyInstaller will create folder 'dist/UltiMaker-Cura' where all files will be. Now you can test if it is working
```
dist/UltiMaker-Cura/UltiMaker-Cura
```

Next we need also to add cura.desktop-file and cura-icon.png before building AppImage.

#### Download appimagetool for aarch64
```
cd dist/
wget https://github.com/AppImage/appimagetool/releases/download/continuous/appimagetool-aarch64.AppImage
```

#### Copy cura icon
```
cp ~/Development/Cura/resources/images/cura-icon.png UltiMaker-Cura/
```

#### Create desktop entry
```
nano UltiMaker-Cura/cura.desktop
```

```
[Desktop Entry]
Name=UltiMaker Cura
Name[de]=UltiMaker Cura
GenericName=3D Printing Software
GenericName[de]=3D-Druck-Software
GenericName[nl]=3D-Print Software
Comment=Cura converts 3D models into paths for a 3D printer. It prepares your print for maximum accuracy, minimum printing time and good reliability with many extra features that make your print come out great.
Exec=UltiMaker-Cura %F
Icon=cura-icon
Terminal=false
Type=Application
MimeType=model/stl;application/vnd.ms-3mfdocument;application/prs.wavefront-obj;image/bmp;image/gif;image/jpeg;image/png;text/x-gcode;application/x-amf;application/x-ply;application/x-ctm;model/vnd.collada+xml;model/gltf-binary;model/gltf+json;model/vnd.collada+xml+zip;
Categories=Graphics;
Keywords=3D;Printing;
X-AppImage-Version=5.7.0
Path=
StartupNotify=false
```

#### Create AppRun link to point UltiMaker-Cura executable
```
ln -sf UltiMaker-Cura/UltiMaker-Cura UltiMaker-Cura/AppRun
```

#### Build AppImage
```
ARCH=aarch64 ./appimagetool-aarch64.AppImage UltiMaker-Cura
```

That's it! Now you can copy/move AppImage to somewhere and use .desktop file to launch AppImage!
#### Don't forget to change line 'Exec=UltiMaker-Cura %F' in .desktop file to point to your AppImage!
