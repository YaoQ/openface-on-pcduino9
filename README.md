# Openface installation on pcDuino9

This is the tutorial to introduce the openface and torch installation on pcDuino9(RK3288) which is running Debian 8.1(Jessie).
 
## 1. Download openface source code
```
git clone https://github.com/cmusatyalab/openface
cd openface
git submodule init
git submodule update 
./models/get-models.sh 
```

## 2. Install dependencies 
```bash
sudo apt-get update 
sudo apt-get install build-essential cmake curl gfortran libatlas-dev libavcodec-dev  libavformat-dev  libboost-all-dev  libgtk2.0-dev libjpeg-dev liblapack-dev  libswscale-dev  pkg-config python-dev python-pip  wget  zip libpng16-16 -y
sudo apt-get clean && sudo rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
sudo pip install cython
sudo pip2 install numpy 
sudo pip2 install scipy 
sudo pip2 install pandas
sudo pip2 install scikit-learn
sudo pip2 install scikit-image
sudo apt-get install python-skimage
```

## 3. Install torch

### a. Install deps
* You can download this [updated script](https://github.com/YaoQ/openface-on-pcduino9/blob/master/install-deps-debian)
* Run the script
```
bash install-deps-debian
```

Note: if you get error like **mismatch**
    * change the apt source:
    `vim /etc/apt/sources.list` 
    
        	deb http://httpredir.debian.org/debian jessie main contrib non-free
        	deb-src http://httpredir.debian.org/debian jessie main contrib non-free
    
    * then clean and rerun install-deps
    ```
    sudo rm -rf /var/lib/apt/lists/* 
    sudo rm -vf /var/lib/apt/lists/partials/* 
    ```

### b. Install torch
```
git clone https://github.com/torch/distro.git ~/torch --recursive
cd ~/torch && ./install.sh
source ~/.bashrc
```
Note: at this point, torch installation path has been installed in ~/.bashrc. 

```
. <your home path>/torch/install/bin/torch-activate
export PYTHONPATH=$PYTHONPATH:<your home path>/openface:<your home path>/openface/openface
```

### c. Update the environment
```
sudo ~/torch/install/bin/luarocks install nn
sudo ~/torch/install/bin/luarocks install dpnn
sudo ~/torch/install/bin/luarocks install optim
sudo ~/torch/install/bin/luarocks install csvigo
```

At this point, the command-line program **th** should be available in your shell.

## 4. Install opencv
* Install deps
```
sudo apt-get -y install libopencv-dev build-essential checkinstall cmake pkg-config yasm libtiff5-dev libjpeg-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libdc1394-22-dev libxine2-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libv4l-dev
sudo apt-get -y install python-dev python-numpy libtbb-dev libqt4-dev libgtk2.0-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev x264 v4l-utils
```
* Download opencv source code
```
wget -O OpenCV-2.4.11.zip http://sourceforge.net/projects/opencvlibrary/files/opencv-unix/2.4.11/opencv-2.4.11.zip/download
```
* Compile Opencv
```
unzip OpenCV-2.4.11.zip
cd opencv-2.4.11
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D WITH_OPENMP=ON -D WITH_FFMPEG=OFF ../
make -j4
sudo make install
sudo sh -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
sh -c 'echo "export PYTHONPATH=/usr/local/lib/python2.7/site-packages:\$PYTHONPATH"  >> ~/.bashrc'
source ~/.bashrc
```

## 5. Install dlib
For openface 0.2, we need dlib 19.1 version.
* Go to:`https://sourceforge.net/projects/dclib/`
* Download the file and unzip it. 
```
wget https://sourceforge.net/projects/dclib/files/latest/download -O dlib.tar.bz2 
tar -xvf dlib.tar.bz2
cd dlib/python_examples
mkdir build
cd build
cmake ../../tools/python -DUSE_SSE4_INSTRUCTIONS=ON
# The following requires CPU later than 2011
＃cmake ../../tools/python －DUSE_AVX_INSTRUCTIONS=ON
cmake --build . --config Release
sudo cp dlib.so /usr/local/lib/python2.7/dist-packages
```
**th** needs to exit the current terminal, and open a new one, then we can run **th**.
```
sudo pip install txaio
sudo pip install autobahn
sudo pip install  imagehash
sudo pip install  imutils
sudo pip install matplotlib
```

## 6. Fix the models for ARM architecture
* Download ascii model
```bash
wget http://openface-models.storage.cmusatyalab.org/nn4.small2.v1.ascii.t7.xz
unzx nn4.small2.v1.ascii.t7.xz
mv nn4.small2.v1.ascii.t7 openface/models/openface
```
* Update the file main.lua to point the new model file
`vim batch-represent/main.lua`
change:
	`model = torch.load(opt.model)`
to:
	`model = torch.load('models/openface/nn4.small2.v1.ascii.t7','ascii')`
* Update the openface_server.lua file
`vim openface/openface_server.lua`
Change:
	`net = torch.load(opt.model)`
to: 
`net = torch.load('./models/openface/nn4.small2.v1.ascii.t7','ascii')`

## 7. Run openface Demos
### a. Compare
```
cd ~/openface
./demos/compare.py images/examples/{lennon*,clapton*}
Comparing images/examples/lennon-1.jpg with images/examples/lennon-2.jpg.
  + Squared l2 distance between representations: 0.782
Comparing images/examples/lennon-1.jpg with images/examples/clapton-1.jpg.
  + Squared l2 distance between representations: 1.059
Comparing images/examples/lennon-1.jpg with images/examples/clapton-2.jpg.
  + Squared l2 distance between representations: 1.170
Comparing images/examples/lennon-2.jpg with images/examples/clapton-1.jpg.
  + Squared l2 distance between representations: 1.402
Comparing images/examples/lennon-2.jpg with images/examples/clapton-2.jpg.
  + Squared l2 distance between representations: 1.552
Comparing images/examples/clapton-1.jpg with images/examples/clapton-2.jpg.
  + Squared l2 distance between representations: 0.379

```
### b. Run Server
```
./openface/demos/web/start-servers.sh
```
