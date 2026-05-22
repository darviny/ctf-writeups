---
description: voice-in-the-packet, loud-packets, check-the-fine-prints
---

# \[tjctf2026] forensics

## check-the-fine-prints

```shellscript
check-the-fine-prints/logo.png
---------------------------------------------------------------------------------------------------
DECIMAL                            HEXADECIMAL                        DESCRIPTION
---------------------------------------------------------------------------------------------------
0                                  0x0                                PNG image, total size: 14276 
                                                                      bytes
14276                              0x37C4                             ZIP archive, file count: 
                                                                      248, total size: 50741 bytes
---------------------------------------------------------------------------------------------------

```

`binwalk -e logo.png`  extracts the files hidden inside.

001.png - 248.png are extracted.

