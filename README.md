Compile Android7.1 industry firmware (Ubuntu 12.04 64bit)

# Requirements

Ubuntu 12.04.5
  * ubuntu-12.04.5-desktop-amd64.iso

Android 7.1 SDK
  * rk3399-firefly-industry-71-20190926.7z.001 (8,589,934,592 bytes)
  * rk3399-firefly-industry-71-20190926.7z.002 (4,017,872,526 bytes)

Android 7.1 SDK Update
  * rk3399-industry-nougat-bundle.7z (1,470,510,364 bytes)

To compile Android, requirement of PC is:
  * 64 bit CPU
  * 16 GB physical memory + swap memory
  * 30 GB free disk space to build. The source code tree take another 8 GB space.\
     => 100 GB free disk space to builds. The source code tree take another 13 GB space. 

Minimum requirements :
  * CPU - 4 Cores
  * RAM - 16 GB physical memory + swap memory
  * Disk - 120 GB

# Prerequisites
Install OpenJDK 8:
```
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

Install Git 2.19.2:
```
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
```

Install 7zip:
```
sudo apt-get install p7zip-full
```

Install devel Libraries:
```
sudo apt-get install git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386  g++-multilib mingw32 tofrodos gcc-multilib ia32-libs python-markdown libxml2-utils xsltproc zlib1g-dev:i386 lzop libssl1.0.0 libssl-dev
```

Install Cross compiler:
```
sudo apt-get install gcc-arm-linux-gnueabihf  lzop libncurses5-dev  libssl1.0.0 libssl-dev
```

Install Tools for NDK:
```
sudo apt-get install texinfo wine bison flex dmake libtool pbzip2
```

Set environmental variables:
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
```

Extract Android 7.1:
```
mkdir -p ~/proj/firefly-rk3399-Industry
cd ~/proj/firefly-rk3399-Industry
7z x /path/to/rk3399-firefly-industry-71-20190926.7z.001
git reset --hard
```

Update Android 7.1:
```
cd ~/proj/firefly-rk3399-Industry
7z x /path/to/rk3399-industry-nougat-bundle.7z -r -o.
mv rk3399-industry-nougat-bundle/ .bundle/
.bundle/update
git rebase FETCH_HEAD
```

# Android source code compile error: "Try increasing heap size with java option '-Xmx<size>'"
```
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
jack-admin kill-server && jack-admin start-server
```

# Android 7.1 mkimage.sh fail on Ubuntu 12.04 

Patch ~/proj/firefly-rk3399-Industry/device/rockchip/common/sparse_tool.py
```
@@ -46,7 +46,7 @@
  *   *   *   *   *   *  % (total_blks, blk_sz, total_chunks))
  *   *   *   *  out_total_chunks = 0
  *   *   *   *  out_head = bytearray(28)
-  *   *   *   * out_chunk = bytearray(12)
+  *   *   *   * out_chunk = struct.pack("<2H2I", 0,0,0,0)
  *   *   *   *  chunk_struct = list(struct.unpack("<2H2I", out_chunk))
  *   *   *   *  buffer = bytearray(align_unit * 1024)
  *   *   *   *  offset = 0
```

# Use Firefly’s script to compile

Only compile the kernel:
```
cd ~/proj/firefly-rk3399-Industry
./FFTools/make.sh -k -j8
```
Only compile the Uboot:
```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh -u -j8
```
Only compile the Android:
```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh -a -j8
```
Compile Ubooot, Kernel, Android:
```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh -j8
```

# Product Firefly-RK3399 complie

Default Compilation ： HDMI+DP
```
./FFTools/make.sh -j8
./FFTools/mkupdate/mkupdate.sh
```

Compile EDP7.85
```
./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_box-userdebug
```

# Manual compilation

Compile Kernel:
```
cd ~/proj/firefly-rk3399/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-firefly.img
```

Compile Uboot:
```
cd ~/proj/firefly-rk3399/u-boot/
make rk3399_box_defconfig
make ARCHV=aarch64 -j8
```

Compile Android:
```
cd ~/proj/firefly-rk3399/
source build/envsetup.sh
lunch rk3399_firefly_box-userdebug
make -j8
./mkimage.sh
```

# Create update.img

Compiled with Firefly’s script can be packaged into update.img, run: ./FFTools/mkupdate/mkupdate.sh update After the package is finished, the update.img will be generated under rockdev/Image-rk3399_firefly_box/ Create update.img in Windows is simple. Just copy the files to AndroidTool’s rockdev\Image directory as previous step. Then run the batch file mkupdate.bat in rockdev directory, which will create update.img under rockdev\Image.


# Flashing partition images

./mkimage.sh at previous step will repack boot.img and system.img, and copy other related image files to the rockdev/Image-rk3288/ directory. The common image files are listed below:

* boot.img : Android’s initramfs, to initialize and mount system partition.
* kernel.img : Kernel image.
* misc.img : Misc partition image, to switch boot mode and pass parameter in recovery mode.
* recovery.img : Recovery mode image.
* resource.img : Resource image, containing boot logo and kernel’s device tree info.
* system.img : System partition image with ext4 filesystem format.
* trust.img ：File about sleep
* RK3399MiniLoaderAll_V1.05.bin ：Loader
* uboot.img ：uboot
