---
title: "[EN] Bitbake cheat sheet"
date: 
draft: true

# cover:
#     image: images/20181204A_banniere.png
#     alt: "cover"
#     relative: true

# tags: ["français"]

---





## Modify the Linux kernel with configuration fragments in Yocto

Modifying the kernel configuration in Yocto may be different depending on the original conditions.

In general, would be convenient to adopt configuration fragments that can be generated directly with the Bitbake command executing the following steps:

1. Prepare for the kernel configuration (take a snapshot)

```bash
bitbake -c kernel_configme virtual/kernel
```

2. Edit the configuration (do your modifications)

```bash
bitbake -c menuconfig virtual/kernel
```

3. Save the configuration differences (extract the differences)

```bash
bitbake -c diffconfig virtual/kernel
```

The differences will be saved in $WORKDIR/fragment.cfg

You have to copy the $WORKDIR/fragment.cfg in your layer directory and in the recipe (bbappend) as in the example below. 

