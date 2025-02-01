# Freedreno mesa kgsl install guide
Add bookworm-backports:
```
echo -e "deb [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://deb.debian.org/debian bookworm main contrib non-free\n\
deb [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://deb.debian.org/debian bookworm-updates main contrib non-free\n\
deb [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://security.debian.org/debian-security bookworm-security main contrib non-free\n\
deb [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://deb.debian.org/debian bookworm-backports main contrib non-free\n\
deb-src [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://deb.debian.org/debian bookworm main contrib non-free\n\
deb-src [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://deb.debian.org/debian bookworm-updates main contrib non-free\n\
deb-src [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://security.debian.org/debian-security bookworm-security main contrib non-free\n\
deb-src [signed-by=\"/usr/share/keyrings/debian-archive-keyring.gpg\"] http://deb.debian.org/debian bookworm-backports main contrib non-free" | tee /etc/apt/sources.list > /dev/null
```

Install dependencies:
```
apt update && apt upgrade 
apt install wet git
apt -t bookworm-backports build-dep mesa
apt install llvm
```
Download patches and mesa:
```
cd ~
git clone https://github.com/xMeM/termux-packages.git -b dev/mesa --single-branch
wget https://archive.mesa3d.org/mesa-24.2.6.tar.xz
tar xf mesa-24.2.6.tar.xz
```
Apply patches:
```
cd mesa-24.2.6
PATCH_DIR=/root/termux-packages/packages/mesa; for PATCH in $(ls -1 $PATCH_DIR | grep .patch); do [[ $PATCH == "0008"* ]] && echo "SKIP PATCH !!!: $PATCH" || patch -p1 < $PATCH_DIR/$PATCH; done
```
Configure, build and install mesa:
```
meson build -Dcpp_rtti=true -Dgbm=enabled -Dopengl=true -Degl=enabled -Degl-native-platform=x11 -Dgles1=disabled -Dgles2=enabled -Ddri3=enabled -Dglx=dri -Dllvm=enabled -Dshared-llvm=disabled -Dplatforms=x11,wayland -Dosmesa=true -Dglvnd=enabled -Dxmlconfig=disabled -Dgallium-drivers=swrast,virgl,zink,freedreno -Dvulkan-drivers=swrast,freedreno -Dfreedreno-kmds=msm,kgsl --prefix=/usr --libdir=/usr/lib/aarch64-linux-gnu --reconfigure
ninja -C build install
```
