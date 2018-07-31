# ExeFS Patches

ExeFS patches for various Nintendo Switch sysmodules, intended to be loaded by [Atmosphère-loader](https://github.com/Atmosphere-NX/Atmosphere/tree/master/stratosphere/loader).

## Installation

First, make sure you're using Atmosphère's [loader reimplementation](https://github.com/Atmosphere-NX/Atmosphere/tree/master/stratosphere/loader).
This is most commonly done via [hekate](https://github.com/CTCaer/hekate).

Download this repository as a .zip file using the green "Clone or download" button found above the file listing.
To install all the patches, you can extract it to to the root of your microSD card. Alternatively, you can copy
over only the patches you are interested in.

## Patch List & Compatibility

| Patch Name | 1.0.0 | 2.0.0 | 2.1.0 | 2.2.0 | 2.3.0 | 3.0.0 | 3.0.1 | 3.0.2 | 4.0.0 | 4.0.1 | 4.1.0 | 5.0.0 | 5.0.1 | 5.0.2 | 5.1.0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [disable_ca_verification](#disable-ca-verification) | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| [am_dev_function](#am-dev-function) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

### Disable CA Verification

**Thanks to:** [SciresM](https://github.com/SciresM)

**Affected Sysmodules:** ssl

Disables certificate authority verification on SSL requests.

### AM Dev Function

**Thanks to:** [misson20000](https://github.com/misson20000)

**Affected Sysmodules:** am

Forces the check for `am.debug!dev_function` setting to always pass. This allows use of [CreateSelfLibraryAppletCreatorForDevelop](https://reswitched.github.io/SwIPC/ifaces.html#nn::am::service::IAllSystemAppletProxiesService(400)).
