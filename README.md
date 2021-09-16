# ExeFS Patches

ExeFS patches for various Nintendo Switch sysmodules, intended to be loaded by [Atmosphère-loader](https://github.com/Atmosphere-NX/Atmosphere/tree/master/stratosphere/loader).

## Installation

First, make sure you're using Atmosphère's [loader reimplementation](https://github.com/Atmosphere-NX/Atmosphere/tree/master/stratosphere/loader).
This is most commonly done via [hekate](https://github.com/CTCaer/hekate).

Download this repository as a .zip file using the green "Clone or download" button found above the file listing.
To install all the patches, you can extract it to to the root of your microSD card. Alternatively, you can copy
over only the patches you are interested in.

### Disable CA Verification

**Thanks to:** [SciresM](https://github.com/SciresM) (original patches), [Adubbz](https://github.com/Adubbz) (porting assistance), [misson20000](https://github.com/misson20000) (porting), [tesnos](https://github.com/tesnos) (porting), [simontime](https://github.com/simontime) (porting), [thomasnet](https://github.com/thomasnet-mc) (porting), and [Behemoth](https://github.com/HookedBehemoth) (porting).

**Affected Titles:** ssl

Disables certificate authority verification on SSL requests made by sysmodules.

**Available Versions:**
| Version | Build ID |
| --- | --- |
| 1.0.0 | 22EB12F05B9AC3CEF3E207350E388E85D4495FDB |
| 2.0.0 | 0A6CD7F6CF67BEB4DC9FFFC999EB37A80A788793 |
| 3.0.0 | 71EB62FE5E28B959CBC2BD4BD0AAB1E384837C8B |
| 3.0.1 | 41510266E1CF6CAC72300CE717B31180232EEEED |
| 4.0.0 | F19352C1E0D9E8EA14F44D7780C0737D7FD6818B |
| 5.0.0 | 12FB8206DEBF729341ECE612A3F7BC3E66FA104E |
| 5.1.0 | 637B1B1640D2F99BA79C0BF89C736DCF464B5649 |
| 6.0.0 | CCE1AF6F242C7638C829ADA19B10E83FF4215DC4 |
| 7.0.0 | 46B0C69D8A9D9FDDEC0A88F2EAAB0F9D516516F1 |
| 8.0.0 | 92B0F02D173C81F2A4E217DE8402258F1F497433 |
| 8.1.0 | BE9001211DCC7C46833726B1A5EE343E7CDFE3CD |
| 9.0.0 | CEC9F20754ED3B1EFA21CCB474CA211A54C6F6D4 |
| 10.0.0 | D4FF6C2AB6E573B664965E7DC742B2071566D5B0 |
| 11.0.0 | 93B46B1A8ADD9FF5CC9A4A31C4A6070662C001FE |
| 12.0.0 | 095E912CCC7E37448437DAE6BDD5702D6CC9AAE9 |

### Disable Browser CA Verification

**Thanks to:** [thomasnet](https://github.com/thomasnet-mc), RTNX, and [amicuchu](https://github.com/amicuchu) (porting)

**Affected Titles:** BrowserDll

Disables certificate authority verification on SSL requests made by browsers.

**Available Versions:**
| Version | Module ID |
| --- | --- |
| 8.0.0 | 5C9D10FEEC39A84B1E4BD101E7D62418 |
| 8.1.0 | 439C3D3CE513FEBB4961B5EE605A3A9C |
| 9.0.0 | DC3F7C8B0209AAB0CED63CC67A180C76 |
| 9.0.1 | 7BB88FB3C9F58F575A02B043D9F1123B |
| 9.1.0 | E561A756E05BDD08A9F69684D81E4EAF |
| 10.0.0 | FB7DD6BF69A43C98F80DB368C0C9D43D |
| 11.0.0 | 49C5C9021F3FABD2F5535385666AB7DC |
| 12.0.0 | FA7EA8512C4FF7059F5D4101268FA4C0 |

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

**Available Versions:**
| Version | Build ID |
| --- | --- |
| 1.0.0 | C79F22F18169FCD3B3698A881394F6240385CDB1 |
| 2.0.0 | 7C7B8934F26C93BCA6DE78536B373A6C93BB4BD0 |
| 2.1.0 | BE419FD6F1BA24BF5543D94C9A68CAC4C9A34FF1 |
| 3.0.0 | 01890C643E9D6E17B2CDA77A9749ECB9A4F676D6 |
| 3.0.1 | C088ADC91417EBAE6ADBDF3E47946858CAFE1A82 |
| 4.0.0 | 3EC573CB22744A993DFE281701E9CBFE66C03ABD |
| 5.0.0 | B37E82B05DE4921265EF0AD37D90236A0748551D |
| 5.1.0 | B95EF6B3C808B177002B1F1DEBB8ABBAFAA013B8 |
| 6.0.0 | A6CCE73F370E6E3CAED0F92ECA4D1B50DC6F1763 |
| 6.1.0 | A4AB4FCCF62A4B78A8C2E83759F3343D38C91E89 |

### Fatal Force Extra Info

**Thanks to:** [misson20000](https://github.com/misson20000)

**Affected Sysmodules:** fatal

Forces the check for `fatal!show_extra_info` setting to always pass. This causes the fatal error screen to show the title id, aslr base, and a backtrace of the process that caused the fatal error state.
This patch works as an alternative to overwriting the setting via `set:fd`.

**Available Versions:**
| Version | Build ID |
| --- | --- |
| 5.0.0 | 3EE4F9B74776C5C2320971B15035F6D2DB6E2A9D |

### vi Debug

**Thanks to:** [Behemoth](https://github.com/HookedBehemoth)

**Affected Sysmodules:** vi

Ignores debug settings value to enable `caps:sc` 1201-1203 and 1204 for 9.0.0-9.2.0.

**Available Versions:**
| Version | Build ID |
| --- | --- |
| 3.0.0 | C41FC90700B2E371A88EA96D8E688E9AC954E40F |
| 3.0.1 | CDD21E75CDEC621583FAE6A09B7E64A4789BCD37 |
| 4.0.0 | FC5A4D49AA96C39EFEECECAAB3A4338C89A0151F |
| 5.0.0 | 7B4123290DE2A6F52DE4AB72BEA1A83D11214C71 |
| 5.1.0 | 723DF02F6955D903DF7134105A16D48F06012DB1 |
| 6.0.0 | 967F4C3DFC7B165E4F7981373EC1798ACA234A45 |
| 7.0.0 | 98446A07BC664573F1578F3745C928D05AB73349 |
| 8.0.0 | 0767302E1881700608344A3859BC57013150A375 |
| 8.1.0 | 7C5894688EDA24907BC9CE7013630F365B366E4A |
| 9.0.0 | 7421EC6021AC73DD60A635BC2B3AD6FCAE2A6481 |
| 10.0.0 | 96529C3226BEE906EE651754C33FE3E24ECAE832 |
| 11.0.0 | D689E9FAE7CAA4EC30B0CD9B419779F73ED3F88B |
| 11.0.1 | 65A23B52FCF971400CAA4198656D73867D7F1F1D |
| 12.0.0 | B295D3A8F8ACF88CB0C5CE7C0488CC5511B9C389 |
| 13.0.0 | 82EE58BEAB54C1A9D4B3D9ED414E84E31502FAC6 |

### CapSrv Debug

**Thanks to:** [Behemoth](https://github.com/HookedBehemoth)

**Affected Sysmodules:** capsrv

Ignores global debug flag to enable DeleteAlbumFileByAruidForDebug and LoadMakerNoteInfoForDebug.

**Available Versions:**
| Version | Build ID |
| --- | --- |
| 9.0.0 | 95AA853B87297A9C00D05A1C9658B5261359E6F6 |
| 10.0.0 | CE10E3973F600CA89CB4F4A387CE5DFFE19F777F |
| 11.0.0 | B2905698AB812AF271161DA11ECFD624B5E9E0E7 |
| 12.0.0 | 3EF8BB918AB693AB8B64DEF74B506EC7B205D442 |
| 12.1.0 | DA400E5F50C6CEDB7F7B949D717C45C83B9B6DD7 |
| 13.0.0 | 5A692E929D0EE5C7D7A71E2E228BDC22FBEDEB27 |

### Nifm CTest

**Thanks to:** [Behemoth](https://github.com/HookedBehemoth)

**Affected Sysmodules:** nifm

Skips connection test to allow connections to networks without internet access or dns wildcard blocks (e.g. \*nintendo\*).

**Available Versions:**
| Version | Build ID |
| --- | --- |
| 1.0.0-7 | 88E0743F72954F95E6D33B37507FCEA7FC6BCC34 |
| 1.0.0 | D47D1506009B340829CD545B2A3F3AA7881FBADA |
| 2.0.0 | E6BFDADD5C69E17D43B7C67E2B2EE8B2E50C8E1F |
| 2.1.0 | AD32FB6D8F36668C586E538E32576A8D6A3931C0 |
| 3.0.0 | 881206E6B5078EFC4E2C30D7B33E33AD266538C6 |
| 3.0.1 | 5835E2DADB4DD570DD811ABF521FA91AC3C7B717 |
| 4.0.0 | 38774C42DFCB8B9D7AA61550D6AF7D335472556C |
| 5.0.0 | B6966381A806655B718F1BF11DB5FF836E3085F7 |
| 5.1.0 | D82361E0D66DC01AFA3B5116532E5E1ED569C578 |
| 6.0.0 | 3AED90979B380C6415F975F5B784BEA2B4730E8C |
| 6.1.0 | 43F10952AE80CFADC39A0BF59EA4E552EF4A4528 |
| 7.0.0 | 929014BFCFE462FD76B2BB3454FB304F63C73AC2 |
| 8.0.0 | FD53CB863709DFFEC19C0889F61D4C424AFFD4ED |
| 9.0.0 | 7DF07326B6B50CA37F19C1C44F9458406C536B30 |
| 9.1.0 | 9C72C47F0310F7C9F487047D8EB42DBB96882088 |
| 10.0.0 | 5DA461C7B6CAE6B88EDF4F914F7CBCF0943B10BB |
| 11.0.0 | 7A43F840337C28D453718843608EEFF78AFD460B |
| 12.0.0 | A85F50FBA10E06A3EBA3D3FACB9E075B218C7D6D |
| 12.1.0 | 69E25CDEEED5C6520AC2AC8E5EAE01CD8FC46E40 |
| 13.0.0 | 8CB532EA199207191F04CE3DDECEC854C7CF07D6 |
