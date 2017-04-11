titile: compile cm for galaxy nexus
categories: cyanogenmod
tags: [Android, cyanogenmod]
---
为galaxy nexus编译cm-10.2的步骤

### 下载代码
1. Download Source

        repo init -u https://github.com/CyanogenMod/android.git -b cm-10.2
        repo sync -j 1

2. Get prebuilt apps(CM11 and below)

        cd vendor/cm
        ./get-prebuilts
 
3. Prepare the device-specific code

        source build/envsetup.sh
        breakfast maguro
   
4. Extract proprietary blobs   
进行这个步骤需要连上galaxy nexus

        cd device/samsung/maguro
        ./extract-files.sh   

### 编译
    brunch maguro

### 参考
[https://wiki.cyanogenmod.org/w/Build_for_maguro](https://wiki.cyanogenmod.org/w/Build_for_maguro)