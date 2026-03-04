---
title: Налаштування мінімального Debian 13 (чи 12) із XFCE4
categories: [Linux]
---
Шпаргалка про встановлення Debian з графічним інтерфейсом XFCE та мінімальним потрібним мені набором додатків. Перевірено із Debian 12 та 13.

При розгортанні ОС, на етапі вибору додаткових програм обрати лише SSH сервер. Після встановлення:
```bash
ssh username@address

sudo -i

if   grep bookworm /etc/os-release; then
    ntp_prog="ntp"
elif grep trixie   /etc/os-release; then
    ntp_prog="ntpsec"
fi \
&&  apt update \
&&  apt upgrade -y \
&&  apt install -y \
    $ntp_prog \
    libxfce4ui-utils \
    thunar \
    xfce4-appfinder \
    xfce4-panel \
    xfce4-session \
    xfce4-settings \
    xfce4-terminal \
    xfconf \
    xfdesktop4 \
    xinit \
    network-manager-gnome \
    firefox-esr firefox-esr-l10n-uk \
    ristretto \
    xfce4-screenshooter \
    xfce4-whiskermenu-plugin \
    xfce4-xkb-plugin \
    xfce4-power-manager-plugins \
    xfce4-panel-profiles python3-gi \
    xfce4-pulseaudio-plugin \
    thunar-archive-plugin \
    cups \
    system-config-printer \
    mousepad \
    cifs-utils \
    parole  \
    gimp \
    libreoffice-writer libreoffice-calc libreoffice-impress libreoffice-gtk3 libreoffice-l10n-uk \
&& sed -r -i 's/^deb(.*)$/deb\1 contrib/g' /etc/apt/sources.list \
&& apt update \
&& apt install -y ttf-mscorefonts-installer \
&& sed -i '/^#greeter-hide-users=false/s/^#//' /etc/lightdm/lightdm.conf \
&& sed -i -e 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=1/g' /etc/default/grub \
&& update-grub \
&& sudo reboot

ssh username@address

wget -P /tmp/ https://anykey.cv.ua/files/debian-xfce/xfce4-custom-panel
xfce4-panel-profiles load /tmp/xfce4-custom-panel
```
