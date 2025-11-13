# UHD Installation

To use devices that require the **UHD** driver, such as the **USRP B210** radio, it is essential to follow the installation steps in this section.

This process compiles and installs the UHD driver (v4.8.0.0):

```bash
# [https://files.ettus.com/manual/page_build_guide.html](https://files.ettus.com/manual/page_build_guide.html)

#1. Install UHD dependencies
sudo apt install -y autoconf automake build-essential ccache cmake cpufrequtils \
doxygen ethtool g++ git inetutils-tools libboost-all-dev libncurses-dev \
libusb-1.0-0 libusb-1.0-0-dev libusb-dev python3-dev python3-mako \
python3-numpy python3-requests python3-scipy python3-setuptools python3-ruamel.yaml

#2. Clone the UHD repository
git clone [https://github.com/EttusResearch/uhd.git](https://github.com/EttusResearch/uhd.git) ~/uhd
cd ~/uhd

#3. Checkout the specific version (v4.8.0.0)
git checkout v4.8.0.0

#4. Build and install
cd host
mkdir build
cd build
cmake ../
make -j $(nproc)
make test # This step is optional

#5. Install and load the libraries
sudo make install
sudo ldconfig

#6. Download firmware/FPGA images
sudo uhd_images_downloader
