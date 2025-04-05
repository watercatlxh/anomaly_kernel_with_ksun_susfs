### Anomaly Kernel Build Guide  
After some attempts, I found that building directly in the kernel source code is not feasible.  
In fact, it should be run in the AOSP source code to produce a viable kernel.  
Many thanks to  
https://xdaforums.com/t/kernel-anomaly-kernel-for-oneplus-8t-a14-15-03-12-25-custom.4704735/ The_Anomalist for their help  
https://github.com/The-Anomalist/Anomaly-Kernel/ Anomaly Kernel source code  
https://github.com/JackA1ltman/NonGKI_Kernel_patches JackAltman's kernel patches  

### Build Guide  
This kernel is based on crDroid, so you should first clone the crDroid repository.  

### Installing Dependencies and Repo  
Several packages are needed to build crDroid:  
```bash
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick lib32ncurses-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses6 libncurses-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```  
(During my testing, `libwxgtk3.2-dev` from the Ubuntu official repository was no longer installable, so I removed it. This doesn’t seem to affect the build.)  

### Install Repo Tool  
```bash
# Make a directory where Repo will be stored and add it to the path  
mkdir ~/bin  
PATH=~/bin:$PATH  

# Download Repo itself  
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo  

# Make Repo executable  
chmod a+x ~/bin/repo  
```  

### 1.2 Initializing Repo  
```bash
# Create a directory for the source files  
# This can be located anywhere (as long as the filesystem is case-sensitive)  
mkdir crDroid  
cd crDroid  

# Install Repo in the created directory  
repo init -u https://github.com/crdroidandroid/android.git -b 15.0 --git-lfs --depth=1  
```  
(Due to the large size of the original repository, I added `--depth=1` to reduce disk usage and speed up cloning. If you don’t need it, you can remove this option.)  

This is what you will run each time you want to pull in upstream changes. Keep in mind that on your first run, it is expected to take a while as it will download all the required Android source files and their change histories.  

```bash
# Let Repo take care of all the hard work  
repo sync  

# Run to prepare our devices list  
source build/envsetup.sh  

# Since the OnePlus device codename is "kebab," I run this  
breakfast kebab  
```  
Now your build environment is ready.  

You can now edit `.repo/local_manifests/roomservice.xml` to replace your kernel. For example:  
```xml
<project  
    name="The-Anomalist/Anomaly-Kernel"  
    path="kernel/oneplus/sm8250"  
    remote="github"  
    revision="Echelon" />  
```  
Then force a sync:  
```bash
repo sync -j$(nproc) --force-sync kernel/oneplus/sm8250  
```  

After completion, you can switch to the kernel folder and apply the KernelSU susfs patch.  

### Patch KernelSU and susfs (Optional)  
```bash
cd kernel/oneplus/sm8250  
echo "CONFIG_KSU=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_HAS_MAGIC_MOUNT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_SUS_PATH=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_SUS_MOUNT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_AUTO_ADD_SUS_KSU_DEFAULT_MOUNT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_AUTO_ADD_SUS_BIND_MOUNT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_SUS_KSTAT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "# CONFIG_KSU_SUSFS_SUS_OVERLAYFS is not set" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_TRY_UMOUNT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_AUTO_ADD_TRY_UMOUNT_FOR_BIND_MOUNT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_SPOOF_UNAME=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_ENABLE_LOG=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_HIDE_KSU_SUSFS_SYMBOLS=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_SPOOF_CMDLINE_OR_BOOTCONFIG=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KSU_SUSFS_OPEN_REDIRECT=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  

# KSUN  
curl -LSs "https://raw.githubusercontent.com/rifsxd/KernelSU-Next/next-susfs/kernel/setup.sh" | bash -s next-susfs-dev  

# KSU  
curl -LSs "https://raw.githubusercontent.com/tiann/KernelSU/main/kernel/setup.sh" | bash -s main  

# MKSU  
curl -LSs "https://raw.githubusercontent.com/ShirkNeko/KernelSU/main/kernel/setup.sh" | bash -s susfs-dev  
```  
(KSUN has been tested and compiled successfully.)  

### Apply Patches  
```bash
git clone https://github.com/JackA1ltman/NonGKI_Kernel_Patches.git -b op_kernel NonGKI_Kernel_Patches --depth=1  
git clone https://gitlab.com/simonpunk/susfs4ksu.git -b kernel-4.19 susfs4ksu --depth=1  
cp susfs4ksu/kernel_patches/fs/* fs/  
cp susfs4ksu/kernel_patches/include/linux/* include/linux/  
```  

Thanks to JackAltman's patches:  
```bash
git clone https://github.com/watercatlxh/anomaly_kernel_with_ksun_susfs.git  
bash anomaly_kernel_with_ksun_susfs/Patches/normal_patches.sh  

echo "CONFIG_COMPAT_VDSO=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_LTO_NONE=y" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_SHADOW_CALL_STACK=n" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_RELR=n" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
echo "CONFIG_KASAN_STACK_ENABLE=n" >> ./arch/arm64/configs/vendor/kona-perf_defconfig  
sed -i 's/# CONFIG_SECCOMP is not set/CONFIG_SECCOMP=y/' ./arch/arm64/configs/vendor/kona-perf_defconfig  

cp susfs4ksu/kernel_patches/50_add_susfs_in_kernel-$KERNEL_VERSION.patch ./  
patch -p1 < 50_add_susfs_in_kernel-${{ env.KERNEL_VERSION }}.patch || true  

cp NonGKI_Kernel_Patches/kona_oos/namespace_fixed.patch ./  
cp NonGKI_Kernel_Patches/kona_oos/task_mmu_fixed.patch ./  
cp NonGKI_Kernel_Patches/kona_oos/ksu_backport.patch ./  

patch -p1 < namespace_fixed.patch || true  
patch -p1 < task_mmu_fixed.patch || true  
patch -p1 < ksu_backport.patch || true  
```  

Return to the crDroid root directory and run:  

```bash
make bootimage -j$(nproc)  
```  
You will now have a perfect kernel image!  