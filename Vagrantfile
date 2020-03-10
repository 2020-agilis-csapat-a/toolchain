YAY_REPO_URL = 'https://aur.archlinux.org/yay.git'
GETTY_OVERRIDE_PATH = '/etc/systemd/system/getty@tty1.service.d'

INSTALL_DEV_PACKAGES = <<-END
  pacman -S --needed --noconfirm git base-devel
END

INSTALL_PYTHON = <<-END
  pacman -S --needed --noconfirm python python-pip
END

INSTALL_YAY = <<-END
  pushd .
    if [ ! -e /tmp/yay ]; then
      git clone '#{YAY_REPO_URL}' /tmp/yay/
    fi
    cd /tmp/yay
    makepkg --clean
    makepkg --noconfirm -si
  popd
END

INSTALL_PYCHARM = <<-END
  yay -S --needed --noconfirm pycharm-professional
END

# Or we could just, you know...
# use SSH and (Neo)Vim or Emacs?
INSTALL_GUI = <<-END
  pacman --noconfirm --needed -S xfce4 \
    xorg-server xf86-input-libinput
  yes | pacman -S --needed virtualbox-guest-utils
  systemctl daemon-reload
  systemctl enable --now vboxservice.service
END

SETUP_GUI = <<-END
  echo 'startxfce4' > ~/.xinitrc
END

AUTOLOGIN_OVERRIDE = <<-END
[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --autologin vagrant --noclear %I $TERM
END

AUTO_GUI_PROFILE = <<-END
  grep -q 'tty' ~vagrant/.bash_profile || \
    echo '[[ "$(tty)" = "/dev/tty1" ]] && startx' >> ~vagrant/.bash_profile
END

AUTOLOGIN_SYSTEMD = <<-END
  mkdir -p '#{GETTY_OVERRIDE_PATH}'
  echo '#{AUTOLOGIN_OVERRIDE}' > '#{GETTY_OVERRIDE_PATH}/override.conf'
  systemctl daemon-reload
END

RESTART_TTY = <<-END
  systemctl restart getty@tty1.service
END

SETUP_PARALLEL_MAKEPKG = <<-END
  sed -i '/etc/makepkg.conf' -e 's/^COMPRESSXZ=.*$/COMPRESSXZ=\(xz -c -z - --threads=0\)/'
  sed -i '/etc/makepkg.conf' -e 's/#MAKEFLAGS=.*$/MAKEFLAGS=\(-O2 -j3 -march=native)/'
END

VM_NAME = 'agile-dev'

Vagrant.configure('2') do |config|
  config.vm.box = 'archlinux/archlinux'
  config.vm.box_version = '2020.03.04'

  config.vm.define VM_NAME
  config.vm.hostname = VM_NAME

  config.vm.provider :virtualbox do |vm|
    vm.name = VM_NAME
    vm.gui = true

    # Everyone can spare that much, right?
    vm.cpus = 2
    vm.memory = 2048
  end

  config.vm.provision :shell, privileged: true, inline: 'pacman -Sy --noconfirm'
  config.vm.provision :shell, privileged: true, inline: INSTALL_DEV_PACKAGES
  config.vm.provision :shell, privileged: true, inline: INSTALL_PYTHON
  config.vm.provision :shell, privileged: true, inline: INSTALL_GUI

  config.vm.provision :shell, privileged: true, inline: SETUP_PARALLEL_MAKEPKG

  config.vm.provision :shell, privileged: false, inline: INSTALL_YAY
  config.vm.provision :shell, privileged: false, inline: INSTALL_PYCHARM

  config.vm.provision :shell, privileged: false, inline: SETUP_GUI
  config.vm.provision :shell, privileged: false, inline: AUTO_GUI_PROFILE
  config.vm.provision :shell, privileged: true, inline: AUTOLOGIN_SYSTEMD

  config.vm.provision :shell, privileged: true, inline: RESTART_TTY
end
