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

## Install leptonica dependencies
```
sudo yum install zlib
sudo yum install zlib-devel
sudo yum install libjpeg
sudo yum install libjpeg-devel
sudo yum install libwebp
sudo yum install libwebp-devel
sudo yum install libtiff
sudo yum install libtiff-devel
sudo yum install libpng
sudo yum install libpng-devel
```

## Install leptonica from source
```
cd /home/asureuser
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
wget https://github.com/tesseract-ocr/tesseract/archive/4.0.0.tar.gz -O tesseract-4.0.0.tar.gz
tar xvvfz tesseract-4.0.0.tar.gz
cd tesseract-4.0.0
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
./autogen.sh
./configure --prefix=/usr/local --disable-shared --enable-static --with-extra-libraries=/usr/local/lib/
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

# Load tesseract languages
Create a new location to store the trained language data and then export that location as the `TESSDATA_PREFIX`
```
mkdir -p /home/azureuser/tess/traineddata
```
Then add this path to your `~/.bash_profile` file.
```
export TESSDATA_PREFIX=/home/azureuser/tess/traineddata
export TESSDATA_PREFIX
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
