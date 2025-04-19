# semantic bugs

### 1.Introduction

This is a semantic bug and will result in some unexpected behaviors.

Specifically, 7Z is expected to notify errors when handling a broken xz file according the [specification][1] (e.g., it will report errors if CRC check fails). However, we find a bug that 7Z fails to report the corrupted file and returns OK. We attach the corrupted file under the directory `pocs`.

An attacker can exploit the bug to construct a malicious compressed xz file. 7Z will return OK (which is unexpected) when handling the file. As a result, the actual error is ignored and remains silent.

*Potential Threat:* Assuming a situation, an important program relies on 7Z to handle some compressed components, and it will fail in handling such files but returns OK. Finally, when it needs the components in some critical tasks, it may be too late to avoid the unexpected behavior.

### 2.Trigger the bug

+ Running environment
	* OS: Ubuntu 22.04 or Ubuntu 18.04

+ Target program required
	* We need to download the target program, and there are two options: 1) download & compile the source code from the [homepage][2] 2) use the command `sudo apt install 7zip`. Option2 is easy and more convenient. 

+ Steps

	+ Using xz util to show the file is corrupted, using the command
		```
		xz -d pocs/poc1.xz
		```
		The xz util reports `xz: poc1.xz: Unsupported options`, the error message indicates the file is broken and can not be decompressed.
	+ Using 7Z to show it fails to output the error message
		```
		7zz e -so pocs/poc1.xz
		```
	+ Root cause analysis
	
		After some manual analysis, we locate its root cause is related to parsing `Block Flags`, and it is expected to indicate an error according to the [specification][1]. We quote the related sentence:
		`"If any reserved bit is set, the decoder MUST indicate an error"`. Obviously, 7Z reports nothing, which is unexpected and violates the specification `MUST indicate an error`. The corresponding function in xz util parser is `lzma_block_header_decode`, which correctly identifies the error.

### 3.Notes
It is the same procedure to trigger the second bug (but with a different input file `pocs/poc2.xz`). The reason why I apply for two different CVEs is because they have different root cause. For the second bug, its root cause is related to parsing `stream flags`. The corresponding function in xz util parser is `lzma_stream_header_decode`, which correctly identifies the error. 



[1]: https://tukaani.org/xz/xz-file-format.txt
[2]: https://www.7-zip.org/download.html

CVE-2022-47111 is assigned for this.
