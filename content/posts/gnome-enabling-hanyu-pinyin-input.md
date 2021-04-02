---
title: "Gnome: Enabling Hanyu Pinyin Input"
tags: ["linux", "gnome", "ibus", "pinyin"]
date: 2021-04-02T11:29:56-04:00
---

Or: how to reset your Gnome configuration with
[`gsettings`](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/desktop_migration_and_administration_guide/configuration-overview-gsettings-dconf)

<!-- more -->

I've been learning Chinese for a while on Duolingo, but I had some trouble
getting my computer running [Arch Linux](https://archlinux.org/) with
[Gnome](https://www.gnome.org/) to let me type Chinese characters using [Hànyǔ
Pīnyīn](https://en.wikipedia.org/wiki/Pinyin)

I tried to add Pinyin input support using the method listed as Hanyu Pinyin
(with AltGr dead keys) in Gnome's Region & Language > Input Sources settings,
but switching my keyboard layout to Hanyu Pinyin with Super+Space seemed to
do...  nothing.

A little googling later, I realized I needed to install Pinyin support for
[ibus](https://wiki.archlinux.org/index.php/IBus), so I installed
[ibus-pinyin](https://archlinux.org/packages/community/x86_64/ibus-pinyin/) and
[ibus-libpinyin](https://archlinux.org/packages/community/x86_64/ibus-libpinyin/)
and tried to add Pinyin support, only to see that Gnome still knew only about
the old Hanyu Pinyin+AltGr dead key layout that didn't work for me.

More googling later, I realized I had to restart `ibus` for it to notice that
I'd added more language support:

```
$ ibus restart
```

At this point I also did a full system update, and then when I went back to add
the Pinyin keyboard layout, the Gnome Settings app started to crash when I
clicked the `+` button to add an input source. What was further, the crash
seemed persistent: the Settings app continued to crash after a reboot. I found
the crash trace with [`journalctl
-b`](https://www.freedesktop.org/software/systemd/man/journalctl.html), which
looked like this:

```
systemd-coredump[2997]: Process 2944 (gnome-control-c) of user 1000 dumped core.

Stack trace of thread 2944:
#0  0x00007fc1f1f6f9fe __strcmp_avx2 (libc.so.6 + 0x15c9fe)
#1  0x000056285fad993c n/a (gnome-control-center + 0xcb93c)
#2  0x000056285fada4a9 cc_input_chooser_new (gnome-control-center + 0xcc4a9)
#3  0x000056285fad1105 n/a (gnome-control-center + 0xc3105)
#4  0x00007fc1f3a3c83b g_cclosure_marshal_VOID__OBJECTv (libgobject-2.0.so.0 + 0x1183b)
#5  0x00007fc1f3a5b096 g_signal_emit_valist (libgobject-2.0.so.0 + 0x30096)
#6  0x00007fc1f3a5b210 g_signal_emit (libgobject-2.0.so.0 + 0x30210)
#7  0x00007fc1f327aeeb n/a (libgtk-3.so.0 + 0x1d3eeb)
#8  0x00007fc1f312caf7 n/a (libgtk-3.so.0 + 0x85af7)
#9  0x00007fc1f3a5b096 g_signal_emit_valist (libgobject-2.0.so.0 + 0x30096)
#10 0x00007fc1f3a5b210 g_signal_emit (libgobject-2.0.so.0 + 0x30210)
#11 0x00007fc1f323b3ad n/a (libgtk-3.so.0 + 0x1943ad)
#12 0x00007fc1f3a426ec g_cclosure_marshal_VOID__BOXEDv (libgobject-2.0.so.0 + 0x176ec)
#13 0x00007fc1f3a5b096 g_signal_emit_valist (libgobject-2.0.so.0 + 0x30096)
#14 0x00007fc1f3a5b210 g_signal_emit (libgobject-2.0.so.0 + 0x30210)
#15 0x00007fc1f32328f8 n/a (libgtk-3.so.0 + 0x18b8f8)
#16 0x00007fc1f323abbb n/a (libgtk-3.so.0 + 0x193bbb)
#17 0x00007fc1f323b7f4 n/a (libgtk-3.so.0 + 0x1947f4)
#18 0x00007fc1f3201303 gtk_event_controller_handle_event (libgtk-3.so.0 + 0x15a303)
#19 0x00007fc1f33dbbcf n/a (libgtk-3.so.0 + 0x334bcf)
#20 0x00007fc1f312b235 n/a (libgtk-3.so.0 + 0x84235)
#21 0x00007fc1f3a5b096 g_signal_emit_valist (libgobject-2.0.so.0 + 0x30096)
#22 0x00007fc1f3a5b210 g_signal_emit (libgobject-2.0.so.0 + 0x30210)
#23 0x00007fc1f33ea5a5 n/a (libgtk-3.so.0 + 0x3435a5)
#24 0x00007fc1f3286705 n/a (libgtk-3.so.0 + 0x1df705)
#25 0x00007fc1f3287a5d gtk_main_do_event (libgtk-3.so.0 + 0x1e0a5d)
#26 0x00007fc1f2fe4a03 n/a (libgdk-3.so.0 + 0x3da03)
#27 0x00007fc1f3034a64 n/a (libgdk-3.so.0 + 0x8da64)
#28 0x00007fc1f3949f9c g_main_context_dispatch (libglib-2.0.so.0 + 0x53f9c)
#29 0x00007fc1f399da49 n/a (libglib-2.0.so.0 + 0xa7a49)
#30 0x00007fc1f39476f1 g_main_context_iteration (libglib-2.0.so.0 + 0x516f1)
#31 0x00007fc1f3b5222e g_application_run (libgio-2.0.so.0 + 0xcd22e)
#32 0x000056285fa5c2dc main (gnome-control-center + 0x4e2dc)
#33 0x00007fc1f1e3ab25 __libc_start_main (libc.so.6 + 0x27b25)
#34 0x000056285fa5c4fe _start (gnome-control-center + 0x4e4fe)
```

Some googling later, I found this helpful issue:
https://bugzilla.redhat.com/show_bug.cgi?id=1933397, which supplies a
workaround:

```
$ gsettings set org.gnome.desktop.input-sources show-all-sources false
```

With the workaround, I could add Chinese (Pinyin) input and I can finally type
wise words in simplified Chinese using pinyin:

我的猫很忙 (wǒ de māo hěn máng) viz. my cat is busy
