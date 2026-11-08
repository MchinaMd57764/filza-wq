# filza-wq

A repackaged **Filza** file manager for **stock (non-jailbroken) iOS 27.0.x** that can browse
system containers which are normally out of reach for App Store apps.

The app opens on an **Accessible Roots** hub and lets you browse, read, copy and delete files
inside these locations on your own device:

| Hub entry | Target |
|---|---|
| ConfigurationProfiles | `/var/containers/Shared/SystemGroup/systemgroup.com.apple.configurationprofiles/Library/ConfigurationProfiles` |
| MobileGestalt Cache | `/var/containers/Shared/SystemGroup/systemgroup.com.apple.mobilegestaltcache/Library/Caches` |
| System Data | `/var/containers/Data/System` |
| AppGroup Containers | `/var/mobile/Containers/Shared/AppGroup` |
| My Documents | The app's own Documents folder |

Paths outside the accessible roots redirect back to the hub, so the file manager never gets
stuck in a directory it cannot list.

## How it works

Filza itself is unchanged except for one injected dylib
(`Frameworks/FilzaApplySandboxExt.dylib`). The dylib uses the publicly documented
**`worse_query`** container-query sandbox-escape technique: it asks `libsystem_containermanager`
for a container of the app's declared App Group, swaps `Library/Caches` for a symlink to the
target, and consumes the sandbox extension token that `container_copy_sandbox_token` returns.
This is a repackage of a public technique, not an original discovery.

## Requirements

* **iOS 27.0.x** — developed and tested on iOS 27.0.1 (24A446), iPhone 16 Pro Max.
  Later iOS versions may have changed the container-query behavior; no guarantee.
* **A signing identity that supports App Groups.** The IPA is **unsigned** and must be re-signed
  on your side, for example with 轻松签 / ESign / Sideloadly / your own developer certificate.
  The escape only works if the signature carries the App Groups entitlement:

  ```xml
  <key>com.apple.security.application-groups</key>
  <array>
      <string>group.com.filza.wq</string>
  </array>
  ```

  Use `group.com.filza.wq` or any other App Group that your provisioning profile allows.
  The tweak reads the group from the signature at runtime, so the name only has to be
  consistent with what you sign with.

* Free Apple IDs cannot provision App Groups. Use a paid Apple Developer account or a
  signing service/certificate that supports the App Groups capability.

## Install

1. Download [`Filza-wq.ipa`](Filza-wq.ipa).
2. Sign it with your certificate:
   * 轻松签 / ESign: import the IPA, enable **App Groups** in the signing options and make sure
     the App Groups entitlement includes a group your profile allows (e.g. `group.com.filza.wq`).
   * Sideloadly / Xcode: sign with a provisioning profile that contains the App Group.
3. Install the signed IPA and trust the developer certificate on the device.

## Usage

* Launch the app. It lands on **Accessible Roots**.
* Tap a shortcut to browse the corresponding location; use the back button to return.
* Deleting items works as in Filza. Deleted items go to `Documents/.Trash` inside the app
  container and can be moved back manually.
* Renaming, copying and creating folders are also available from Filza's menus.

## Notes and disclaimer

* This is a **research / educational** build. Use it only on devices you own.
* Be careful: deleting system container contents can break system settings, configuration
  profiles and app data. There is no undo for files deleted outside the app's Trash folder.
* Provided as-is, without warranty of any kind.
* Not affiliated with Apple Inc. or TIGI Software.

## Credits

* [Filza File Manager](https://tigisoftware.com/) by TIGI Software — the file manager UI and
  all original functionality.
* The container-query sandbox-escape technique published by **[@forcequitOS](https://x.com/forcequitOS)**
  and **[@0xjohnny](https://x.com/0xjohnny)** (public `worse_query` project) — this repository
  only packages their public technique into a Filza build for research purposes. All credit for
  the technique belongs to them.
