# 安裝加速器(方式1)

- 修改user-rootfsconfig檔 位於`proj-dict/project-spec/meta-user/conf/`
  
  - 新增 CONFIG_accelerator 
  
  - 新增 CONFIG_rc.d
    
    > CONFIG_accelerator 安裝加速器
    > 
    > CONFIG_rc.d 開機時，執行一個script檔，以便取代原來的kv260-d 這個空加速器 

- 執行 petalinux-config -c rootfs 後，選擇新增的項目
  
  > Filesystem Package --->   
  > Petalinux Package Groups  --->
  > Image Features  --->
  > apps   ---> 
  > 
  > >               [*] accelerator   
  > >               [ ] gpio-demo
  > >               [ ] peekpoke 
  > >               [*] rc.d
  > 
  > User packages   --->
  > PetaLinux RootFS Settings  --->     

- 新增二個目錄 位於`proj-dict/project-spec/meta-user/recipes-apps/`
  
  - 新增 accelerator目錄
  
  - 新增 rc.d目錄

- 進入 `proj-dict/project-spec/meta-user/recipes-apps/accelerator`
  
  - 創建新檔名為 `accelerator.bb` 編輯如下:
    
    ```bash
    #
    # This file is the ntpdate recipe.
    #
    #  
    
    SUMMARY = "my aceelerator"
    SECTION = "PETALINUX/apps"
    LICENSE = "MIT"
    LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
    
    SRC_URI = "file://kv260_8K15P.dtbo \
               file://kv260_8K15P.bit.bin \
               file://shell.json    \                    
        "
    
    S = "${WORKDIR}"
    # 新增目錄
    FILES_${PN} += "/lib/firmware/xilinx/kv260_8K15P"
    
    do_install() {
                  install -d ${D}/lib/firmware/xilinx/kv260_8K15P
                 install -m 0755 ${S}/kv260_8K15P.dtbo ${D}/lib/firmware/xilinx/kv260_8K15P
                 install -m 0755 ${S}/kv260_8K15P.bit.bin ${D}/lib/firmware/xilinx/kv260_8K15P
                 install -m 0755 ${S}/shell.json ${D}/lib/firmware/xilinx/kv260_8K15P
    }  
    ```
  
  - 創建目錄 `files` 將編譯出來的加速器檔放至此目錄中 
    
    - kv260_8K15P.dtbo
    
    - kv260_8K15P.bit.bin
    
    - shell.json

- 進入`proj-dict/project-spec/meta-user/recipes-apps/rc.d`
  
  - 創建新檔名為 `rc.d.bb` 編輯如下:
    
    ```bash
    #
    # This file is the rc.d recipe.
    #
    
    SUMMARY = "Simple rc.d application"
    SECTION = "PETALINUX/apps"
    LICENSE = "MIT"
    LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
    
    inherit update-rc.d
    
    SRC_URI = "file://replaced8K15P.sh \
        "
    RDEPENDS_${PN} += "bash"
    
    S = "${WORKDIR}"
    INITSCRIPT_PARAMS = "defaults 10"
    INITSCRIPT_NAME = "replaced8K15P.sh"
    
    do_install() {
             install -d ${D}/etc/init.d
             install -m 0755 ${S}/replaced8K15P.sh ${D}/etc/init.d
    }
    ```
  
  - 創建目錄 `files` 並新增一個`replaced8K15P.sh` script file編輯如下:
    
    ```bash
    #!/bin/bash
    
    sudo xmutil unloadapp
    sudo xmutil loadapp kv260_8K15P
    ```
    
    > Note: `先卸載空白加速器再載入設計的加速器`

# 安裝加速器(方式2)

- 在專案目錄裡輸入
  
  `petalinux-create -t apps --name accelerator --enable`
  
  `petalinux-create -t apps --name rc.d --enable`
  
  即可創建目錄及Enable創建的選項

- 陸續編輯方式1中的檔案即可
