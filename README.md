# ELAN 04f3:0c7f fingerprint sensor on Arch / CachyOS

A PKGBUILD that builds [jdtanner's experimental libfprint driver](https://github.com/jdtanner/elan0c7f-libfprint-driver) for the **ELAN 04f3:0c7f** fingerprint sensor (found in Acer Swift and similar laptops) on Arch-based distros, including ones that ship **OpenCV 5** (like CachyOS).

All the real work — reverse-engineering the sensor and writing the driver — is jdtanner's. This repo only packages it for Arch and fixes a few build problems.

> ⚠️ **Experimental.** The driver is new, matching is done in software, and it misses sometimes (around 60% first-try matches on my laptop). The driver's author doesn't recommend it for login yet. Use it for the lock screen, keep your password, and don't use it for `sudo`.

## Do I have this sensor?

```
lsusb | grep -i 04f3:0c7f
```

If you see `Elan Microelectronics Corp. ELAN:Fingerprint`, and `fprintd-list $USER` says **No devices available**, this is for you.

## Install

1. (Optional but smart) make a snapshot if you use snapper:
   ```
   sudo snapper -c root create -d "before fingerprint driver"
   ```
2. Install the build tools:
   ```
   sudo pacman -S --needed base-devel git meson gtk-doc gobject-introspection opencv fprintd
   ```
3. Build and install:
   ```
   git clone https://github.com/elenax1d/elan0c7f-cachyos.git
   cd elan0c7f-cachyos
   makepkg -si
   ```
   Say **yes** when it asks to replace `libfprint`.
4. Check the sensor is found:
   ```
   sudo systemctl restart fprintd
   fprintd-list $USER
   ```
   You should see **ELAN 04f3:0c7f Fingerprint Sensor**.
5. Enroll a finger (KDE: System Settings › Users › Configure Fingerprint Authentication, or `fprintd-enroll`). **Press the same way every time**, the way you'll naturally touch it to unlock — moving the finger around during enrollment makes matching worse.
6. Test: `fprintd-verify`

**Tip:** enrolling the same finger 2–3 times under different names (`fprintd-enroll -f right-middle-finger`, …) gives the matcher more samples and improves the hit rate.

## What this PKGBUILD changes

The upstream driver targets Fedora and OpenCV 4. To build on Arch/CachyOS it:

- accepts **OpenCV 5** (`opencv5` pkg-config name) as well as OpenCV 4
- includes `opencv2/features.hpp` instead of `opencv2/features2d.hpp` on OpenCV 5 (module was renamed)
- links only the five OpenCV modules the driver uses (core, imgproc, features, geometry, flann) — OpenCV 5's pkg-config otherwise pulls in `opencv_viz`, which fails to link without `vtk`
- disables the docs build

It installs as its own package (`libfprint-elan0c7f-git`), so normal system updates won't overwrite it.

## Undo

```
sudo pacman -S libfprint
```
(and say yes to replacing `libfprint-elan0c7f-git`), or roll back to your snapshot from the boot menu.

## Work in progress: KDE lock screen listening right after wake

`wip/kde-fingerprint` is an attempt to make the KDE lock screen listen for a finger immediately after the laptop wakes, instead of only after pressing a key. It raises `pam_fprintd`'s 30-second timeout and retries on failure. **Not confirmed to work yet.** Password unlock uses a separate PAM file and isn't affected.

```
sudo install -m 644 wip/kde-fingerprint /etc/pam.d/kde-fingerprint   # try it
sudo rm /etc/pam.d/kde-fingerprint                                    # undo
```

## Tested on

- CachyOS, KDE Plasma 6 (Wayland), fprintd 1.94.5, OpenCV 5.0.0, October 2026
- One laptop. Reports from other machines welcome — open an issue!

## Credits

- Driver: [jdtanner/elan0c7f-libfprint-driver](https://github.com/jdtanner/elan0c7f-libfprint-driver) (LGPL-2.1-or-later)
- libfprint: [freedesktop.org](https://gitlab.freedesktop.org/libfprint/libfprint)
- Lock-screen idea based on [mdouwes/kde-login-fixes](https://github.com/mdouwes/kde-login-fixes)
