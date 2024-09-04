# AArch64 multi-platform
# Maintainer: Jat-faan Wong
# Contributor: Jat-faan Wong, Guoxin "7Ji" Pu, Joshua-Riek 

_panthor_base=aa54fa4e0712616d44f2c2f312ecc35c0827833d
_panthor_branch=rk-6.1-rkr3-panthor
pkgbase=linux-aarch64-rockchip-6.1-joshua-git-rk3588
pkgname=("${pkgbase}"{,-headers})
pkgver=6.1.43.r1266030.gd3e66fee
pkgrel=4
arch=('aarch64')
license=('GPL2')
url="https://github.com/Joshua-Riek"
_desc="with patches picked by Joshua Riek focusing on RK3588" 
makedepends=('cpio' 'xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'dtc')
options=('!strip')
_srcname='linux-rockchip'
source=(
  "git+${url}/${_srcname}.git#branch=noble"
  "config"
  "linux.preset"
  "panthor.patch::https://github.com/hbiyik/linux/compare/${_panthor_base}...${_panthor_branch}.patch"
)

sha512sums=(
  'SKIP'
  'aee1f48667cd853937633c6c965a4ea5ae79e7e36cc1da6e069d7d710e7e0a213978164b4f69e3d4dda3394e1c7cd5e71d8109ab83478a5e7a108116e62fe6f3'
  '60d8c983976d37e218b17511586a316353a8ef14e08477c6d3b5b712d53886617a374b5ea9d2321e1a94c461cf979e6d94cf2c26c3df0da314e53a9223c8329f'
  '9a693c739e90662d2f86c89874638ebba3d75e06c787151f6290d759df92589f4d500c6fe17533c06d2f3437518afaec6488c1c70fb813c7b0b5db9d9d320e20'
)

pkgver() {
  cd "${_srcname}"
  printf "%s.%s%s%s.r%s.g%s" \
    "$(grep '^VERSION = ' Makefile|awk -F' = ' '{print $2}')" \
    "$(grep '^PATCHLEVEL = ' Makefile|awk -F' = ' '{print $2}')" \
    "$(grep '^SUBLEVEL = ' Makefile|awk -F' = ' '{print $2}'|grep -vE '^0$'|sed 's/.*/.\0/')" \
    "$(grep '^EXTRAVERSION = ' Makefile|awk -F' = ' '{print $2}'|tr -d -|sed -E 's/rockchip[0-9]+//')" \
    "$(git rev-list --count HEAD)" \
    "$(git rev-parse --short=8 HEAD)"
}

prepare() {
  cd "${_srcname}"
  
  rm -rf localversion*
  echo "Setting version..."
  echo "-rockchip" > localversion.10-pkgname
  #echo "-r$(git rev-list --count HEAD)" > localversion.20-revision

  # this is only for local builds so there is no need to integrity check. (if needed)
  for p in ../../custom/*.patch; do
    echo "Custom Patching with ${p}"
    patch -p1 -N -i $p || true
  done

  # based on https://github.com/hbiyik/linux-rockchip/tree/noble-panthor
  patch -p1 -N -i ../panthor.patch

  echo "Preparing config..."
  cat "${srcdir}/config" > '.config'
}

build() {
  export ARCH="arm64"
  export CROSS_COMPILE="aarch64-linux-gnu-"
  cd "${_srcname}"

  make prepare
  make -s kernelrelease > version

  unset LDFLAGS
  make ${MAKEFLAGS} Image modules
  make ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

_package() {
  pkgdesc="The ${_srcname} kernel, ${_desc}"
  depends=('coreutils' 'kmod' 'initramfs')
  optdepends=('wireless-regdb: to set the correct wireless channels of your country')

  cd "${_srcname}"
  
  # install dtbs
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs/${pkgbase}" dtbs_install

  # install modules
  make INSTALL_MOD_PATH="${pkgdir}/usr" INSTALL_MOD_STRIP=1 modules_install

  # copy kernel
  local _dir_module="${pkgdir}/usr/lib/modules/$(<version)"
  install -Dm644 arch/arm64/boot/Image "${_dir_module}/vmlinuz"

  # remove reference to build host
  rm -f "${_dir_module}/"{build,source}

  # used by mkinitcpio to name the kernel
  echo "${pkgbase}" | install -D -m 644 /dev/stdin "${_dir_module}/pkgbase"

  # install mkinitcpio preset file
  sed "s|%PKGBASE%|${pkgbase}|g" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the ${_srcname} kernel, ${_desc}"
  depends=("python")

  cd "${_srcname}"
  local builddir="${pkgdir}/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map version
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/arm64" -m644 arch/arm64/Makefile
  cp -t "$builddir" -a scripts

  # add xfs and shmem for aufs building
  mkdir -p "$builddir"/{fs/xfs,mm}

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/arm64" -a arch/arm64/include
  install -Dt "$builddir/arch/arm64/kernel" -m644 arch/arm64/kernel/asm-offsets.s
  mkdir -p "$builddir/arch/arm"
  cp -t "$builddir/arch/arm" -a arch/arm/include

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */arm64/ || $arch == */arm/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#${pkgbase}}
  }"
done
