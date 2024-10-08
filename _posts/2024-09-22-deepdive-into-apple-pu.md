---
layout: post
title:  "Deepdive the Apple M1 Pro"
date:   2024-09-21 09:58:55 -0400
categories: Apple CPU
---
Today I am going to do some more research into the way the Apple GPU works, and maybe verify some of the claims made about its performance. One of the most popular repos for benchmarking is [metal-benchmarks](https://github.com/philipturner/metal-benchmarks), so let's start there and double-check his work. 


| GPU | Gen | GHz | Cores | GOPS32 | GOPS16 | GIPS | L2 $ | L3 $ |
|-----|-----|-------|-------|--------|--------|------|-------|------|
| M1P | 7 | 1.296 | 16 | 5308 | 5308 | 2654 | 256K | 24M |


After a little digging, I found out that the clock, the bit width are all hard coded and everything else is calculated from those values. But let's see if it's possible for us to manually verify some of these claims. Or at least, find the real values for the 14-inch GPU.

The number of cores in the GPU is already exposed in About This Mac -> More Info -> System Report -> Graphics/Displays -> Total Number of Cores. I am going to try to "prove" that there are 14 cores on the GPU even though I already know that there are.

To check for the existence of a GPU, we can write a sanity-check swift file like so
```swift
import MetalKit

guard let device = MTLCreateSystemDefaultDevice() else { 
    fatalError( "Failed to get the system's default Metal device." ) 
}

print("Device: \(device.name)")
```
and compile and run with 
```
swiftc main.swift
./main
```

One of the traps that I fell into was just importing Metal. If you just import Metal, you need to link CoreGraphics library as well, so the compilation script becomes `swiftc main.swift -framework CoreGraphics`

