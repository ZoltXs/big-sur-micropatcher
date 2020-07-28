# big-sur-micropatcher (Version 0.0.17pre)
A primitive USB patcher for installing macOS Big Sur on unsupported Macs

(Note that [ParrotGeek has a Big Sur patcher](https://parrotgeek.com/bigsur/) now; it is an alternative that you should consider.)

Thanks to the following people for their hard work to get Big Sur running on unsupported Macs:

- ASentientBot for developing the Hax series of installer patches which are so incredibly helpful for installing Big Sur on unsupported Macs, as well as for his patch to NVDAResmanTesla.kext which allows the GeForce Tesla (9400M/320M) framebuffer to work in Big Sur.
- jackluke for figuring out how to patch the Recovery USB to bypass compatibility checks and AMFI enforcement in the absence of NVRAM boot-args settings.
- highvoltage12v for modifying IO80211Family.kext so that it can support 802.11n cards on Big Sur.
- ParrotGeek for developing the LegacyUSBInjector kernel extension that allows USB to work on various pre-2011 Mac models.
- testheit for describing how to use a kmutil feature that I was previously unaware of; this turned out to be a good way to make LegacyUSBInjector function under Big Sur.

This documentation is more thorough than for previous versions of this patcher, but it may still be incomplete. Remember that you *do this at your own risk*, you could easily lose your data, expect bugs and crashes, Big Sur is still under development (as is this patcher), etc.

## Compatibility between different releases of this patcher and different Big Sur developer beta releases
- v0.0.16-v0.0.17 (this release): Tested with developer beta 3 (20A5323l), but should be compatible with developer beta 1 and 2 as well.
- v0.0.10-v0.0.15: Tested with developer beta 2 (20A4300b), but should be compatible with developer beta 1 and 3 as well.
- v0.0.9: Tested with developer beta 1 (20A4299v) and 2 (20A4300b), but should be compatible with developer beta 3 as well.
- v0.0.1-v0.0.8: Only tested with and compatible with developer beta 1 (20A4299v). Not compatible with beta 2 and later.

## Compatibility of various Mac models:
Note that this information is incomplete and may not be 100% correct yet, but I'll add more information over time and fix any errors as I learn about them.

Mostly compatible Mac models:
- If you have a 2013 or later Mac, please check [Apple's official list of supported Mac models](https://www.apple.com/macos/big-sur-preview/) (search the page for "See if") first, to make sure that you actually need this patcher.
- By the way, with the exception of Mac Pros, all of the Macs in this section officially support Catalina. This section is basically "Macs without official Big Sur support but with Metal support", with the exception of pre-2012 iMacs that have upgraded GPUs. (In fact, a 2011 iMac with upgraded GPU is almost equivalent to this category. Earlier iMacs may have compatibility problems caused by other components; see below.)
- Late 2013 iMac: Everything should work (and, after step 8, you're finished -- no need for step 9 and later).
- 2010/2012 Mac Pro: I have received positive feedback about this patcher, but I do not know which features work perfectly and which don't. If I had to guess which features might be problematic, I would guess sleep and Wi-Fi. patch-kexts.sh (step 9) should fix Wi-Fi, but I don't know what effect it might have on sleep. (You should upgrade the graphics card, as you would for official compatiblity with macOS Mojave.)
- 2009 Mac Pro: Once it's flashed to MacPro5,1 firmware, it should be equivalent to a 2010/2012 Mac Pro. (As with those, you will want to upgrade to a Mojave-compatible graphics card.)
- Other 2012/2013 Macs: Most things should work after the initial installation, except for Wi-Fi (unless you have upgraded to an 802.11ac Wi-Fi card) or possibly GPU switching (on 15" MacBook Pros). Step 9 of the installation process fixes Wi-Fi support, but GPU switching may not yet be a solved problem.

Partially compatible Mac models:
- Most models that officially support High Sierra, but not Mojave, fall into this category. The exceptions are the Mid 2010 15" and 17" MacBook Pros (see the Incompatible category below), and possibly the Late 2009/Mid 2010 iMacs (see the "Unknown status" category below).
- 2011 Macs: Several features may not work after initial installation (after step 8 finishes), including sleep, screen brightness control, Wi-Fi (unless you have upgraded to an 802.11ac Wi-Fi card), and graphics acceleration (unless you have upgraded the GPU in a 2011 iMac). With the exception of a 2011 iMac with upgraded GPU, no 2011 Mac models have full graphics acceleration under Big Sur at this point in time. Make sure to use the `--2011` command line option in step 9 (or `--2011-no-wifi` if you have upgraded to an 802.11ac Wi-Fi card). This fixes sound and Wi-Fi on all 2011 Macs. On 13" MacBook Pros it also fixes sleep and brightness control, and installs the correct Intel framebuffer driver (this is not actual acceleration but it still increases speed somewhat -- enough to make full-screen YouTube in Safari work with very few frame drops). For 15" and 17" MacBook Pros, disabling the discrete GPU will probably increase performance, and sleep and brightness control probably won't work without disabling it. (That isn't to say that sleep and brightness control will necessarily work even with the discrete GPU disabled -- but it'll probably work if you have a way of disabling the GPU that also keeps sleep and display brightness functional in a dosdude-patched Mojave or Catalina.) For a future patcher release, I will look into adding framebuffer drivers for Radeon HD cards for 2011 iMacs (to get the same quasi-acceleration as 13" MacBook Pros get), but I don't yet know what is possible, and if your iMac does not support graphics acceleration with dosdude's Mojave or Catalina Patchers, then don't expect any improvement in future patcher releases. For 2011 Mac Minis, models with Intel graphics will actually have better graphics performance than models with Radeon graphics, since the former will be able to use the same framebuffer driver that works on the 13" MacBook Pro. MacBook Airs should be equivalent to the 13" MacBook Pro in terms of compatibility with Big Sur and this patcher.
- Late 2009/Mid 2010 white MacBook, 2010 13" MacBook Pro, 2010 MacBook Air, 2010 Mac Mini: In addition to the features which don't work after initial installation on the 2011 13" MacBook Pros (Wi-Fi, sound, graphics acceleration, sleep, display brightness control), Ethernet and USB 1.1 also don't work. The `--all` option for patch-kexts.sh (step 9) installs fixes for Wi-Fi, sound, Ethernet, and USB, as well as drivers that enable the GeForce Tesla (9400M/320M) framebuffer (thereby fixing sleep and display brightness control). However, the installation has to be performed on a newer Mac first (a 2011 or newer Mac, or a 2009 or later Mac Pro), with patch-kexts.sh --all run on that same newer Mac, *then* moved over to the old Mac (via either a hard drive/SSD transplant or by using a USB enclosure or USB hard drive/SSD). Otherwise, the installer will boot because USB 2.0 works, but the keyboard and trackpad won't work because USB 1.1 doesn't work. I intend to improve support for these Mac models in future patcher releases; hopefully it will be possible to patch USB 1.1 support into the installer USB's BaseSystem.dmg.

Incompatible Mac models:
- 2010 15"/17" MacBook Pro: These are crashing in AppleACPICPU.kext during boot. Someone needs to create a patch to fix it, or these Macs will not be running Big Sur.
- Any Macs with a pre-Penryn CPU. Basically, this means the original MacBook Air as well as all 2006/2007 Macs (except for iMacs with upgraded CPUs).

Unknown status:
- Late 2009/Mid 2010 iMac: The best case scenario is that these are comparable to 2011 Macs. The worst case scenario is that they're comparable to the 2010 15"/17" MacBook Pros (in which case they're basically doomed due to the same compatibility problem -- the CPUs are similar, so it's a possibility). Or it could be an in-between situation similar to 2010 white MacBooks. Someone needs to try creating a USB installer, patching it, and booting it (so steps 1, 2, and 4 below), and report back on what happens.
- Macs which have a Penryn CPU but which do not officially support High Sierra: These include pre-2008 iMacs with upgraded CPUs, as well as most 2008 and many 2009 Mac models. All of these require "legacy USB" support, just like (for instance) 2010 white MacBooks. Once support for those MacBooks is improved in a future patcher release, perhaps support for some of these Macs will be worth revisiting.

## Quick instructions for use:

1. Obtain a copy of the macOS Big Sur Developer Preview and use `createinstallmedia` as usual to create a bootable USB stick with the installer and recovery environment, as you would on a supported Mac. This patcher is easier to use if the installer USB stick is not renamed after `createinstallmedia` is used, but it can still work if the USB stick has been renamed (see next step).
2. Download this micropatcher, then run `micropatcher.sh` to patch the USB stick. (If you are viewing this on GitHub, and you probably are, then click "Clone" then "Download ZIP".) If the USB stick has been renamed or micropatcher.sh is otherwise unable to find the USB stick, then try specifying the pathname of the USB stick to micropatcher.sh. The easiest way to do that is to open a Terminal window, drag and drop micropatcher.sh into the Terminal window, go back to Finder, choose Computer from the Go menu, drag and drop the USB stick into the Terminal window, then press Return.
3. Since Disk Utility in Big Sur may have new bugs, this may be a good time to use Disk Utility in High Sierra/Mojave/Catalina to do any partitioning or formatting you may need.
4. Boot from the USB stick.
5. If you need to do any partitioning or formatting with Disk Utility, and you didn't do it in step 5, it's best to do it now.
6. Open Terminal (in the Utilities menu), then run `/Volumes/Image\ Volume/set-vars.sh`. This script will change boot-args and csrutil settings as needed, and also set things up so the Installer will run properly. Don't forget that tab completion is your friend! You can type `/V<tab>/I<tab>/se<tab>` at the command prompt -- that's much less typing! (Run `/Volumes/Image\ Volume/set-vars.sh -v` instead if you want verbose boot, which can be very useful for troubleshooting.)
7. Quit Terminal then start the Installer as you would on a supported Mac.
8. Come back in an hour or two and you should be at the macOS setup region prompt! (If you actually watch the installation process, don't be surprised if it seems to get stuck at "Less than a minute remaining..." for a long time. Allow it well over half an hour. It should eventually reboot on its own and keep going.)
9. If you're on a Late 2013 iMac, or you've replaced the 802.11n card in your 2012/2013 Mac with an 802.11ac card, you're done. Otherwise, press Command-Q and wait a few seconds, then the Setup Assistant should let you shut down. After you shut down, start up again and boot from the patched installer USB again, then open Terminal again. This time, run `/Volumes/Image\ Volume/patch-kexts.sh /Volumes/<name of Big Sur volume>`, for example `/Volumes/Image\ Volume/patch-kexts.sh /Volumes/Macintosh\ HD`. It needs to be the name of the *system* volume. This will patch your Big Sur installation to add working Wi-Fi. (On 2011 MacBook Pro 13" and 2011 MacBook Air, add a "--2011" option after the ".sh" and before the volume name, for example `/Volumes/Image\ Volume/patch-kexts.sh --2011 /Volumes/Macintosh\ HD`, to fix sound, brightness control, and sleep as well as Wi-Fi. If you're going to use the installation on a 2010 or older Mac, add a "--all" option likewise.)
10. If you will be using the Big Sur installation on a different Mac (for instance, installing on a 2011 or later Mac and using it on a 2009 or 2010 Mac), it is possible that the other Mac (the one not used for installation) may try to boot off the wrong APFS snapshot. To prevent this, run zap-snapshots.sh on your System volume, to remove all but the most recent snapshot. For instance, `/Volumes/Image Volume/zap-snapshots.sh /Volumes/Macintosh\ HD`. (Or you can also do this if you are running low on disk space.) This is basically the same as step 9, but with `zap-snapshots` instead of `patch-kexts`, and without any command line options like `--2011` or `--all`.
11. After step 9 (and 10 if necessary), reboot into your Big Sur installation.
12. On 2011 and older Macs, once you have installed Big Sur, make sure to enable Reduce Transparency to eliminate many seemingly random crashes, and if icons on the right-hand side of the menu bar are invisible afterward, try Dark mode. (If you will be using the installation on a 2009/2010 Mac, it would be a good idea to finish the Setup Assistant on a 2011 or later Mac and enable Reduce Transparency before moving the installation over.)
13. (only for 2008 Mac Pros & 2010 or earlier models of other Macs) Before you move the installation over from a newer Mac, you need to set nvram variables. Normally this would be done by the set-vars.sh script in step 6, but you ran that script on a different Mac because the patched installer USB either will not boot on your Mac or will not provide a functioning keyboard/mouse/trackpad on your Mac. To set the nvram variables on the older Mac, first boot into the installer USB or DVD for OS X Mavericks (10.9.5) or older. Yosemite and later will not work! Then open Terminal and run the following commands:

To update from one beta to the next, it should be possible to create a USB for the new beta (using createinstallmedia), patch it with this patcher, then follow the directions above (except skip Disk Utility, i.e. skip steps 3 and 5) to boot from the patched USB and install it on top of the previous beta. (Allow something like 1-3 hours for the update, even if it looks like it froze up at some point, especially on a 2011 or older Mac.) This *will* uninstall the Wi-Fi, etc. kexts that were installed in step 9 on the previous beta, so you will need to redo that step as well. Perhaps someone will find a more convenient update method, but in the meantime, this method should suffice. This method has been tested for updating from beta 1 to beta 2 and from beta 2 to beta 3.

If you want to undo the patcher's changes to boot-args and csrutil settings, then boot from the USB stick, open Terminal, and run `/Volumes/Image\ Volume/reset-vars.sh`. For what it's worth, there is also a script for undoing the kext additions (such as 802.11n Wi-Fi), at `/Volumes/Image\ Volume/unpatch-kexts.sh`. (It is dependent on implementation details of patch-kexts.sh and may not be able to undo kext changes applied through other means. A more time-consuming but more thorough alternative is to reinstall Big Sur on top of your existing installation, as if you were using the method described in the previous paragraph to update to a new Big Sur beta.)

The best way to remove the patch from the USB stick is to redo `createinstallmedia`, but if you are working on patcher development or otherwise need a faster way to do it, you can run `unpatch.sh`.
