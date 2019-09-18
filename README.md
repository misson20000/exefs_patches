# ExeFS Patches

ExeFS patches for various Nintendo Switch sysmodules, intended to be loaded by [Atmosphère-loader](https://github.com/Atmosphere-NX/Atmosphere/tree/master/stratosphere/loader).

## Installation

First, make sure you're using Atmosphère's [loader reimplementation](https://github.com/Atmosphere-NX/Atmosphere/tree/master/stratosphere/loader).
This is most commonly done via [hekate](https://github.com/CTCaer/hekate).

Download this repository as a .zip file using the green "Clone or download" button found above the file listing.
To install all the patches, you can extract it to to the root of your microSD card. Alternatively, you can copy
over only the patches you are interested in.

## Patch List & Compatibility

| Patch Name | Type | 1.0.0 | 2.0.0 | 2.1.0 | 2.2.0 | 2.3.0 | 3.0.0 | 3.0.1 | 3.0.2 | 4.0.0 | 4.0.1 | 4.1.0 | 5.0.0 | 5.0.1 | 5.0.2 | 5.1.0 | 6.0.0 | 6.0.1 | 6.1.0 | 6.2.0 | 7.0.0 | 7.0.1 | 8.0.0 | 8.0.1 | 8.1.0 | 9.0.0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [disable_ca_verification](#disable-ca-verification) | ExeFS | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| [disable_browser_ca_verification](#disable-browser-ca-verification) | NRO | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ | ✓ | ✓ |
| [am_dev_function](#am-dev-function) | ExeFS | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| [fatal_force_extra_info](#fatal-force-extra-info) | ExeFS | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |

### Disable CA Verification

**Thanks to:** [SciresM](https://github.com/SciresM) (original patches), [Adubbz](https://github.com/Adubbz) (porting assistance), [misson20000](https://github.com/misson20000) (porting), [tesnos](https://github.com/tesnos) (porting), and [simontime](https://github.com/simontime) (porting).

**Affected Titles:** ssl

Disables certificate authority verification on SSL requests made by sysmodules.

### Disable Browser CA Verification

**Thanks to:** [thomasnet](https://github.com/thomasnet-mc)

**Affected Titles:** BrowserDll

Disables certificate authority verification on SSL requests made by browsers.

### AM Dev Function

**Thanks to:** [misson20000](https://github.com/misson20000)

**Affected Sysmodules:** am

Forces the check for `am.debug!dev_function` setting to always pass. This allows use of [CreateSelfLibraryAppletCreatorForDevelop](https://reswitched.github.io/SwIPC/ifaces.html#nn::am::service::IAllSystemAppletProxiesService(400)).

#### Porting Notes

Search for `dev_function` string. There should be one xref, to a function that loads various settings and assigns them to global flags.
Find where the `dev_function` flag is written, and change the `and w8, w8, w9` instruction before it to `orr w8, w8, #1` or equivalent.

```
; function listing from 6.0.0
SUB             SP, SP, #0x30
STR             X19, [SP,#0x20+var_10]
STP             X29, X30, [SP,#0x20+var_s0]
ADD             X29, SP, #0x20

; query am.debug
ADRP            X19, #aAmDebug@PAGE ; setting class "am.debug"
ADD             X19, X19, #aAmDebug@PAGEOFF ; "am.debug"
ADRP            X3, #aDevFunction@PAGE ; setting key "dev_function"
ADD             X3, X3, #aDevFunction@PAGEOFF ; "dev_function"

MOV             X0, SP ; output buffer
MOV             W1, #1 ; setting size
MOV             X2, X19 ; setting class
BL              query_setting ; query_setting(&dev_function, 1, "am.debug", "dev_function")
LDRB            W9, [SP,#0x20+var_20] ; load value from output buffer

; compute flag value based off query_setting
; return value and retrieved setting value
CMP             X0, #1
ADD             X0, SP, #0x20+var_1C
CSET            W8, EQ
CMP             W9, #0
CSET            W9, NE

; this is the instruction I patch
ORR             W8, W8, #1 ; Keypatch modified this from:
                        ;   AND W8, W8, W9

; prepare for the next query_setting
ADRP            X9, #g_dev_function_flag@PAGE
ADRP            X3, #aAbortOnSaLost@PAGE ; "abort_on_sa_lost"
ADD             X3, X3, #aAbortOnSaLost@PAGEOFF ; "abort_on_sa_lost"
MOV             W1, #1
MOV             X2, X19

; store the previous setting
STRB            W8, [X9,#g_dev_function_flag@PAGEOFF]
```

### Fatal Force Extra Info

**Thanks to:** [misson20000](https://github.com/misson20000)

**Affected Sysmodules:** fatal

Forces the check for `fatal!show_extra_info` setting to always pass. This causes the fatal error screen to show the title id, aslr base, and a backtrace of the process that caused the fatal error state.
This patch works as an alternative to overwriting the setting via `set:fd`.
