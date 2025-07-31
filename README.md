# server_backup

Бэкапим живой сервер linux при помощи rsync (на примере Ubuntu 22.04):

1) Забираем корневую файловую систему с удаленного хоста по SSH, используя файл фильтра rsync_filter.txt:  
sudo rsync -aAXHvPc \
  -e 'ssh -p {remote_ssh_port} -i {ssh_private_key_for_remote}' \
  --filter="merge rsync_filter.txt" \
  --rsync-path='sudo rsync' \ 
  {remote_user}@{remote_host}:/ {target_for_local_backup}
2) Архивируем скопированную ФС в tar-архив для удобства переноса в целевую ФС:  
  cd {target_for_local_backup} && sudo tar -cvpf ../{archive_name}.tar.gz .
3) Создаём целевую виртуалку. Гипервизор не имеет значения.
4) Размечаем диск по своему желанию, можно прям в Live-CD режиме
5) Включаем в Live-CD режиме SSH, задаём пароль или прокидлываем ключ
6) Копируем получившийся архив из п.2 на Live-CD систему
7) Монтируем разделы диска из п.4 в Live-CD систему, к примеру в /mnt
8) Распаковываем архив в /mnt:  
   sudo tar -xpvf {archive_name}.tar.gz -C /mnt
9) Далее, нужно подмонтировать dev, proc, sys, dev/pts в /mnt и войти в chroot-режим для редактирования fstab:  
    sudo mount --bind /dev /mnt/dev && sudo mount --bind /dev/pts /mnt/dev/pts && sudo mount --bind /proc /mnt/proc && sudo mount --bind /sys /mnt/sys
    chroot /mnt /bin/bash -i
10) Затем, находясь в chroot-е под рутом целевой ФС, редактируем /etc/fstab под размеченный диск. При желании, можно сразу же настроить сеть.
11) После редактирования обновляем GRUB и устанавливаем его на целевой диск (не на разделы):  
    update-grub2
    grub-install /dev/vda
12) Выходим из chroot-а, отмонтируем разделы из п.10:  
    sudo umount /mnt/dev /mnt/dev/pts /mnt/proc /mnt/sys
13) Отмонтируем диск целевой системы от Live-CD системы
14) Перезагружаемся. Если всё сделано правильно, то старая-новая система загрузится в штатном режиме.
