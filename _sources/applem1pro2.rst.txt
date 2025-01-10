Learning which features your Macbook M1 Pro Supports
=====================================================

I want to explore how exactly the Apple M1 Pro chip instruction architecture to learn exactly what I am allowed to expect from my CPU. I eventually want to improve pytorch and tinygrad by helping them with shader kernels, and this is my very first step in getting my hands wet. 

Apple provides beginner information about their ISA on their developer site `Apple ISA docs <https://developer.apple.com/documentation/kernel/1387446-sysctlbyname/determining_instruction_set_characteristics>`_. It is a subset of the features available for all ARM processors, whose features can be found `ARM docs <https://developer.arm.com/documentation/ddi0487/latest>`_, in this 15k page PDF. 

To learn more about the instruction set architecture of the Apple M1 chip, I wrote a small script that outputs whether a features is enabled or not.

.. code-block:: swift

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

A quick introduction to the function ``sysctlbyname``. The signature of the function is:

.. code-block:: swift

    int sysctlbyname(const char * name, void * oldp, size_t * oldlenp, void *newp, size_t newlen);

which allows getting parameters as well as setting them. If you pass in 0 for newlen, you end up just getting the information. If you pass in nil for oldlenp, you end up getting just the size.

In order to learn about the system, we can use ``systemctl -a`` and then observe which system controls are changeable and which aren't. The ones that aren't changeable are properties of the chip itself and not kernel properties. 

.. code-block:: zsh

    #!/bin/zsh

    # Get all sysctl keys
    all_keys=$(sysctl -a 2>/dev/null | cut -d '=' -f1 | sort)

    # Get writable sysctl keys
    writable_keys=$(sysctl -W -a 2>/dev/null | cut -d '=' -f1 | sort)

    # Use comm to find keys in all_keys that are not in writable_keys
    comm -23 <(echo "$all_keys") <(echo "$writable_keys") | while read -r key; do
        echo "$key is not writable."
    done




[Rest of the content follows the same pattern...] 
