---
layout: post
title:  "Learning which features your Macbook M1 Pro Supports"
date:   2024-09-21 09:58:55 -0400
categories: ISA Exploration
---

I want to explore how exactly the Apple M1 Pro chip instruction architecture to learn exactly what I am allowed to expect from my CPU. I eventually want to improve pytorch and tinygrad by helping them with shader kernels, and this is my very first step in getting my hands wet. 

Apple provides beginner information about their ISA on their developer site [here](https://developer.apple.com/documentation/kernel/1387446-sysctlbyname/determining_instruction_set_characteristics). It is a subset of the features available for all ARM processors, whose features can be found [here](https://developer.arm.com/documentation/ddi0487/latest), in this 15k page PDF. 

To learn more about the instruction set architecture of the Apple M1 chip, I wrote a small script that outputs whether a features is enabled or not.
```swift
import Foundation

func checkISAFeature(featureName: String) -> Bool {
    var size = 0
    sysctlbyname(featureName, nil, &size, nil, 0)
    
    var value = 0
    sysctlbyname(featureName, &value, &size, nil, 0)
    
    return value != 0
}

let neonFeature = "hw.optional.neon"
let crc32Feature = "hw.optional.armv8_crc32"

if checkISAFeature(featureName: neonFeature) {
    print("NEON feature is available.")
} else {
    print("NEON feature is not available.")
}

if checkISAFeature(featureName: crc32Feature) {
    print("CRC32 feature is available.")
} else {
    print("CRC32 feature is not available.")
}
```

A quick introduction to the function `sysctlbyname`. The signature of the function is 
```swift 
int sysctlbyname(const char * name, void * oldp, size_t * oldlenp, void *newp, size_t newlen);
```
which allows getting parameters as well as setting them. If you pass in 0 for newlen, you end up just getting the information. If you pass in nil for oldlenp, you end up getting just the size.

In order to learn about the system, we can use `systemctl -a` and then observe which system controls are changeable and which aren't. The ones that aren't changeable are properties of the chip itself and not kernel properties. 

```zsh
#!/bin/zsh

# Get all sysctl keys
all_keys=$(sysctl -a 2>/dev/null | cut -d '=' -f1 | sort)

# Get writable sysctl keys
writable_keys=$(sysctl -W -a 2>/dev/null | cut -d '=' -f1 | sort)

# Use comm to find keys in all_keys that are not in writable_keys
comm -23 <(echo "$all_keys") <(echo "$writable_keys") | while read -r key; do
    echo "$key is not writable."
done
```
I use the following script to figure out which keys are not writable. There are only a few categories that we care about. The category we care about the most is hardware information. I told Claude to provide information about each parameter.
```
hw.activecpu: 8 - Number of active CPU cores
hw.byteorder: 1234 - Byte order (little-endian)
hw.cacheconfig: 8 1 2 0 0 0 0 0 0 0 - Cache configuration details
hw.cachelinesize: 128 - Size of a cache line in bytes
hw.cachesize: 3652796416 65536 4194304 0 0 0 0 0 0 0 - Sizes of various cache levels
hw.cpu64bit_capable: 1 - Indicates if the CPU is 64-bit capable
hw.cpufamily: 458787763 - CPU family identifier (specific to Apple Silicon)
hw.cpusubfamily: 4 - CPU subfamily identifier
hw.cpusubtype: 2 - CPU subtype identifier
hw.cputype: 16777228 - CPU type identifier
hw.ephemeral_storage: 0 - Flag indicating presence of ephemeral storage
hw.features.allows_security_research: 0 - Indicates if security research features are allowed
hw.l1dcachesize: 65536 - Size of L1 data cache in bytes
hw.l1icachesize: 131072 - Size of L1 instruction cache in bytes
hw.l2cachesize: 4194304 - Size of L2 cache in bytes
hw.logicalcpu: 8 - Number of logical CPU cores
hw.logicalcpu_max: 8 - Maximum number of logical CPU cores
hw.memsize: 17179869184 - Total physical memory in bytes
hw.memsize_usable: 16537698304 - Usable physical memory in bytes
hw.ncpu: 8 - Number of physical CPU cores
hw.nperflevels: 2 - Number of performance levels (likely efficiency and performance cores)
hw.optional.* - Various CPU feature flags (e.g., SIMD, ARM features)
hw.packages: 1 - Number of physical CPU packages
hw.pagesize: 16384 - Size of a memory page in bytes
hw.pagesize32: 16384 - Size of a 32-bit memory page in bytes
hw.physicalcpu: 8 - Number of physical CPU cores
hw.physicalcpu_max: 8 - Maximum number of physical CPU cores
hw.serialdebugmode: 0 - Serial debug mode status
hw.targettype: J314s - Target hardware type identifier
hw.tbfrequency: 24000000 - Timebase frequency
hw.use_kernelmanagerd: 1 - Indicates if kernel manager daemon is used
hw.use_recovery_securityd: 0 - Indicates if recovery security daemon is used
```

This is good beginner info, but we really start learning once we get into the optinal features
```
hw.optional.AdvSIMD: 1 - Advanced SIMD (Neon) instructions are supported
hw.optional.AdvSIMD_HPFPCvt: 1 - Advanced SIMD Half-Precision Floating-Point Conversion is supported
hw.optional.arm.FEAT_AES: 1 - AES (Advanced Encryption Standard) acceleration is supported
hw.optional.arm.FEAT_AFP: 0 - Alternate Floating-Point behavior is not supported
hw.optional.arm.FEAT_BF16: 0 - BFloat16 instructions are not supported
hw.optional.arm.FEAT_BTI: 0 - Branch Target Identification is not supported
hw.optional.arm.FEAT_CSV2: 1 - Cache Speculation Variant 2 is supported
hw.optional.arm.FEAT_CSV3: 1 - Cache Speculation Variant 3 is supported
hw.optional.arm.FEAT_DIT: 1 - Data Independent Timing is supported
hw.optional.arm.FEAT_DPB: 1 - Data Persistence (barrier) is supported
hw.optional.arm.FEAT_DPB2: 1 - Data Persistence (barrier) version 2 is supported
hw.optional.arm.FEAT_DotProd: 1 - Dot Product instructions are supported
hw.optional.arm.FEAT_ECV: 0 - Enhanced Counter Virtualization is not supported
hw.optional.arm.FEAT_FCMA: 1 - Floating-point Complex Number Addition and Multiplication is supported
hw.optional.arm.FEAT_FHM: 1 - Floating-point Half-precision Multiply is supported
hw.optional.arm.FEAT_FP16: 1 - Half-precision Floating-point instructions are supported
hw.optional.arm.FEAT_FPAC: 0 - Faulting on AUT instruction with invalid pointer is not supported
hw.optional.arm.FEAT_FRINTTS: 1 - Floating-point to Integer Conversions with Directed Rounding are supported
hw.optional.arm.FEAT_FlagM: 1 - Flag Manipulation instructions are supported
hw.optional.arm.FEAT_FlagM2: 1 - Additional Flag Manipulation instructions are supported
hw.optional.arm.FEAT_I8MM: 0 - Int8 Matrix Multiplication is not supported
hw.optional.arm.FEAT_JSCVT: 1 - JavaScript Conversion instructions are supported
hw.optional.arm.FEAT_LRCPC: 1 - Load-Acquire RCpc instructions are supported
hw.optional.arm.FEAT_LRCPC2: 1 - Load-Acquire RCpc instructions version 2 are supported
hw.optional.arm.FEAT_LSE: 1 - Large System Extensions are supported
hw.optional.arm.FEAT_LSE2: 1 - Large System Extensions version 2 are supported
hw.optional.arm.FEAT_PAuth: 1 - Pointer Authentication is supported
hw.optional.arm.FEAT_PAuth2: 0 - Pointer Authentication version 2 is not supported
hw.optional.arm.FEAT_PMULL: 1 - Polynomial Multiply Long is supported
hw.optional.arm.FEAT_RDM: 1 - Rounding Double Multiply Accumulate is supported
hw.optional.arm.FEAT_RPRES: 0 - Registered Pointer / Return Address Signing is not supported
hw.optional.arm.FEAT_SB: 1 - Speculation Barrier is supported
hw.optional.arm.FEAT_SHA1: 1 - SHA1 cryptographic hash acceleration is supported
hw.optional.arm.FEAT_SHA256: 1 - SHA256 cryptographic hash acceleration is supported
hw.optional.arm.FEAT_SHA3: 1 - SHA3 cryptographic hash acceleration is supported
hw.optional.arm.FEAT_SHA512: 1 - SHA512 cryptographic hash acceleration is supported
hw.optional.arm.FEAT_SME: 0 - Scalable Matrix Extension is not supported
hw.optional.arm.FEAT_SME2: 0 - Scalable Matrix Extension version 2 is not supported
hw.optional.arm.FEAT_SME_F64F64: 0 - SME F64F64 instructions are not supported
hw.optional.arm.FEAT_SME_I16I64: 0 - SME I16I64 instructions are not supported
hw.optional.arm.FEAT_SPECRES: 0 - Speculation Resolution is not supported
hw.optional.arm.FEAT_SSBS: 1 - Speculative Store Bypass Safe is supported
hw.optional.arm.FEAT_WFxT: 0 - WFE and WFI with timeout are not supported
hw.optional.arm.FP_SyncExceptions: 1 - Synchronous Exception Reporting for Floating Point is supported
hw.optional.arm.SME_B16F32: 0 - SME B16F32 instructions are not supported
hw.optional.arm.SME_BI32I32: 0 - SME BI32I32 instructions are not supported
hw.optional.arm.SME_F16F32: 0 - SME F16F32 instructions are not supported
hw.optional.arm.SME_F32F32: 0 - SME F32F32 instructions are not supported
hw.optional.arm.SME_I16I32: 0 - SME I16I32 instructions are not supported
hw.optional.arm.SME_I8I32: 0 - SME I8I32 instructions are not supported
hw.optional.arm64: 1 - ARM64 architecture is supported
hw.optional.armv8_1_atomics: 1 - ARMv8.1 atomic instructions are supported
hw.optional.armv8_2_fhm: 1 - ARMv8.2 FHM (Floating-point Half-precision Multiply) instructions are supported
hw.optional.armv8_2_sha3: 1 - ARMv8.2 SHA3 cryptographic instructions are supported
hw.optional.armv8_2_sha512: 1 - ARMv8.2 SHA512 cryptographic instructions are supported
hw.optional.armv8_3_compnum: 1 - ARMv8.3 Complex Number instructions are supported
hw.optional.armv8_crc32: 1 - ARMv8 CRC32 instructions are supported
hw.optional.armv8_gpi: 1 - ARMv8 Generic Platform Interrupts are supported
hw.optional.breakpoint: 6 - Number of hardware breakpoints supported
hw.optional.floatingpoint: 1 - Floating-point operations are supported
hw.optional.neon: 1 - NEON (Advanced SIMD) instructions are supported
hw.optional.neon_fp16: 1 - NEON half-precision floating-point instructions are supported
hw.optional.neon_hpfp: 1 - NEON half-precision floating-point instructions are supported (alternative name)
hw.optional.ucnormal_mem: 1 - Unaligned access to normal memory is supported
hw.optional.watchpoint: 4 - Number of hardware watchpoints supported
```

We are able to see which instructions our processor natively supports. It turns out we didn't need to write any swift code at all, but we didn't know that at the time, so we accidentally learned a little bit of swift.