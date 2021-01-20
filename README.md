TL;DR
If you just want to use this executable and not go to the trouble of having to build it, please skip ahead to [this section below](https://github.com/second-state/OCR-tesseract-on-Centos7#option-2---use-our-pre-made-executable-that-we-prepared-earlier).

# Installing Tesseract 4.0.0 on Centos7
The goal of this repo is to show how to use a CentOS7 system (**with** root access), to create a static compiled binary which can be copied over to, and used on, a CentOS7 system (**without** root access). 

**Question**: Why would we want to do this?

**Answer**: In some cases you might want to use tesseract on a machine via a cloud provider. For security reasons, specific machines on a specific cloud provider's infrastructure will not allow root access to the remove guest.

**Solution**: What we are about to do is log into a CentOS7 machine **with** root access (this can be any machine i.e. a VM on your local machine etc.). We install all of the dependencies and create sufficuent executables on the machine **with** root access. 

Then, we copy the `tesseract` executable over to the machine **without** root access. We also make sure that the executable is in the path. We also make sure that we have the correct trained language data. Finally we fetch an image and perform the OCR.

# The CentOS7 machine WITH root access 
For this task you can use your own CentOS machine etc. This will have to be a machine on which you have full priveledges.

## Install system dependencies
```
sudo yum install -y git
sudo yum install -y libtool
sudo yum install clang
sudo yum install -y gcc-c++.x86_64
```

## Install leptonica dependencies
```
sudo yum install -y zlib
sudo yum install -y zlib-devel
sudo yum install -y libjpeg
sudo yum install -y libjpeg-devel
sudo yum install -y libwebp
sudo yum install -y libwebp-devel
sudo yum install -y libtiff
sudo yum install -y libtiff-devel
sudo yum install -y libpng
sudo yum install -y libpng-devel
```

## Move executables to /usr/local/lib
```
cd /usr/local/lib
sudo cp /usr/lib64/libjpeg.so.62 .
sudo cp /usr/lib64/libwebp.so.4 .
sudo cp /usr/lib64/libtiff.so.5 .
sudo cp /usr/lib64/libpng15.so.15 .
```

## Install leptonica from source
```
cd /home/azureuser
git clone https://github.com/DanBloomberg/leptonica.git --depth 1
cd /home/azureuser/leptonica
./autogen.sh
./configure --prefix=/usr/local --disable-shared --enable-static --with-zlib --with-jpeg --with-libwebp  --with-libtiff --with-libpng
make
sudo make install
sudo ldconfig
```

## Install tesseract from source
```
cd /home/azureuser
wget https://github.com/tesseract-ocr/tesseract/archive/4.0.0.tar.gz -O tesseract-4.0.0.tar.gz
tar zxvf tesseract-4.0.0.tar.gz
cd tesseract-4.0.0
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
./autogen.sh
./configure --prefix=/usr/local --disable-shared --enable-static --with-extra-libraries=/usr/local/lib/ --with-extra-includes=/usr/local/lib/
make
sudo make install
sudo ldconfig
```

# The CentOS7 machine WITHOUT root access 

## Getting the executable

### Option 1 - Use your own executable (using the procedure above)
If you followed the procedure above, please copy the tesseract binary (from the machine **with** root access) to the Centos7 machine **without** root access. 
```
cd ~
mkdir -p /home/azureuser/tess
cd /home/azureuser/tess
scp -i ~/.ssh/key.pem -rp azureuser@xxx.xx.xxx.xxx:/usr/local/bin/tesseract .
```
Then ensure that the tesseract binary is in the system path on the machine **without** root access, by adding the export statement to the `~/.bash_profile` file
```
export PATH="$PATH:/home/azureuser/tess"
```
Then action those changes
```
source ~/.bash_profile
```

### Option 2 - Use our pre-made executable (that we prepared earlier)
If you just want to use a pre-made executable, you can use wget to fetch it like this. First log into the CentOS7 machine **without** root access. Then fetch the executable like this.
```
cd ~
mkdir -p /home/azureuser/tess
cd /home/azureuser/tess
wget https://github.com/second-state/OCR-tesseract-on-Centos7/raw/main/tesseract.tar.gz
tar -zxvf tesseract.tar.gz
```

Then ensure that the tesseract binary is in the system path on the machine **without** root access, by adding the export statement to the `~/.bash_profile` file
```
export PATH="$PATH:/home/azureuser/tess"
```
Then action those changes
```
source ~/.bash_profile
```

## Finalizing dependencies

### Option 1 - use your own files

Create a new deps directory on the CentOS7 **without** root access
```
mkdir -p /home/azureuser/tess/deps
```

Take the following files from the CentOS7 **with** root access and put them in the `deps` directory that you just created.
```
/usr/lib64/libjbig.so.2.0
/usr/local/lib/libwebp.so.4
/usr/local/lib/libtiff.so.5
/usr/local/lib/libjpeg.so.62
```
Then the following line to the `~/.bash_profile` file on the CentOS7 **without** root access
```
export LD_LIBRARY_PATH="/home/azureuser/tess/deps/"
```
Then action those changes
```
source ~/.bash_profile
```

### Option 2 - use our files (that we prepared earlier)
```
cd /home/azureuser/tess
wget https://github.com/second-state/OCR-tesseract-on-Centos7/raw/main/deps.tar.gz
tar -zxvf deps.tar.gz
```
Then the following line to the `~/.bash_profile` file on the CentOS7 **without** root access
```
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/azureuser/tess/deps/"
```
Then action those changes
```
source ~/.bash_profile
```

# Load tesseract languages
Create a new location to store the trained language data and then export that location as the `TESSDATA_PREFIX`
```
mkdir -p /home/azureuser/tess/traineddata
```
Then add this path to your `~/.bash_profile` file.
```
export TESSDATA_PREFIX=/home/azureuser/tess/traineddata
```
Then action those changes
```
source ~/.bash_profile
```

Copy any of the `.traineddata` files [available here](https://github.com/tesseract-ocr/tessdata) that you need to the above `TESSDATA_PREFIX` location.
```
cd $TESSDATA_PREFIX
wget https://github.com/tesseract-ocr/tessdata/raw/master/fra.traineddata
wget https://github.com/tesseract-ocr/tessdata/raw/master/eng.traineddata
```

# Try tesseract
Grab an image such as [this french lunch menu example]( wget https://second-state.github.io/wasm-learning/faas/ocr/html/a_french_lunch_menu.png)
```
cd /home/azureuser/tess
wget https://second-state.github.io/wasm-learning/faas/ocr/html/a_french_lunch_menu.png
```
Then run the tesseract command and pass in the image as the first parameter 
```
cd /home/azureuser/tess
./tesseract a_french_lunch_menu.png stdout --dpi 70 -l fra
```

# Potential errors
If you get an error relating to a dependency i.e.
```
./tesseract: error while loading shared libraries: xxxxxxx.so.x: cannot open shared object file: No such file or directory
```
Please run `ldd tesseract` as shown below to see which dependencies are `=> not_found` and then place an issue so I can add those to the `deps.tar.gz`
```
[azureuser@centos7-ocr tess]$ ldd tesseract
	linux-vdso.so.1 =>  (0x00007fff2c5bd000)
	libpng15.so.15 => /lib64/libpng15.so.15 (0x00007f55715d6000)
	libjpeg.so.62 (0x00007f5571381000)
	libtiff.so.5 (0x00007f557110d000)
	libwebp.so.4 (0x00007f5570ebd000)
	libz.so.1 => /lib64/libz.so.1 (0x00007f5570ca7000)
	librt.so.1 => /lib64/librt.so.1 (0x00007f5570a9f000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f5570883000)
	libstdc++.so.6 => /lib64/libstdc++.so.6 (0x00007f557057c000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f557027a000)
	libgomp.so.1 => /lib64/libgomp.so.1 (0x00007f5570054000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f556fe3e000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f556fa70000)
	libjbig.so.2.0 => /home/azureuser/tess/deps/libjbig.so.2.0 (0x00007f556f864000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f5571801000)
```
