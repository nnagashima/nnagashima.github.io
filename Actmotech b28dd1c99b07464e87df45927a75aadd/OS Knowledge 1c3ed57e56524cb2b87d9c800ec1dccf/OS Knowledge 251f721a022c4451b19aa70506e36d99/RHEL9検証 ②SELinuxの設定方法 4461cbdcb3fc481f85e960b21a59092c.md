# RHEL9検証 ②SELinuxの設定方法

タグ: RHEL

![images.png](RHEL9%E6%A4%9C%E8%A8%BC%20%E2%91%A6%20Cockpit%E5%B0%8E%E5%85%A5%201289da9f37308067a685deffd613c2bf/images.png)

目次

# 試した環境

---

RHEL9.1

# RHEL9系からSELinuxの設定方法が変わりました

---

従来だと

```
/etc/selinux/config
SELINUX=enforcing
を
SELINUX=disabled
```

にしてOS再起動という流れになりますよね。

RHEL9に関わらず、RHEL9系のOSは今後上記で無効化をしてはいけません。 

そもそも無効化するなとOS作っている側から出ているようです。

じゃあ無効化する方法はないかといったらそうではなく、 RHEL9系の/etc/selinux/configは以下のようになっています。

```
# cat /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
# See also:
# https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-selinux/#getting-started-with-selinux-selinux-states-and-modes
#
# NOTE: In earlier Fedora kernel builds, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
# grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
# grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
# targeted - Targeted processes are protected,
# minimum - Modification of targeted policy. Only selected processes are protected.
# mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

SELINUX の無効化はコメントで書かれているように grubby コマンドで行います。

```
grubby --update-kernel ALL --args selinux=0
```

実施すると/boot/loader/entries/にあるconfにselinux=0が反映されます。

```
# cat /boot/loader/entries/d59b51d40a45461f801c17887f8c5e08-5.14.0-162.6.1.el9_1.0.1.x86_64.conf
title Rocky Linux (5.14.0-162.6.1.el9_1.0.1.x86_64) 9.1 (Blue Onyx)
version 5.14.0-162.6.1.el9_1.0.1.x86_64
linux /vmlinuz-5.14.0-162.6.1.el9_1.0.1.x86_64
initrd /initramfs-5.14.0-162.6.1.el9_1.0.1.x86_64.img
options root=/dev/mapper/rl-root ro resume=/dev/mapper/rl-swap rd.lvm.lv=rl/root rd.lvm.lv=rl/swap selinux=0
grub_users $grub_users
grub_arg --unrestricted
grub_class rocky
```

SELinuxの再有効化は 以下のコマンドになります。

```
grubby --update-kernel ALL --remove-args selinux
```

勿論設定後はOS再起動しないと反映されませんので、ご注意を。

# ためしに/etc/selinux/configで設定してみたら。。。

---

OSが壊れて起動しなくなりました。。。

なので皆さん今後RockyLinux9でSELinuxを触る際には気をつけてください。