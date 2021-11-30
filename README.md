# Gex is an iOS 14.7 jailbreak using CVE-2021-30807 IOMFB exploit

## rest of this readme is from [jsherman212's exploit repo](https://github.com/jsherman212/iomfb-exploit/blob/main/README.md) and probably stuff that is about the exploit and not this jb

To tune for A11 and below, use pongo to load xnuspy and build with
`SAMPLING_MEMORY=1 make -B`. This will enable a test that gathers
the memory returned by `kernel_memory_allocate`, sorts those pointers,
then spits out a range. You'll see something like this:

```
sample_kernel_map: 0xffffffe8ebe9c000 [0x10000 bytes from behind]
sample_kernel_map: to add to alloc_averager:
[0xffffffe8ce934000, 0xffffffe8ebf98000],
```

(just ignore the warnings it spits out)

The test is meant to be ran 30 seconds after the device boots.

Inside `alloc_averager.py` is a couple of samples I already ran for
my phones. It takes the average of all the averages of each range.
Create a "samples list" for your device and add the range to it. 
Repeat the test a couple times until you have 5-10 entries in that
list. `alloc_averager.py` will report a success rate for the guess it
generates based on the list. If you like the success rate, take the guess
and replace the value for `GUESSED_OSDATA_BUFFER_PTR` at the top of
`IOMobileFramebufferUserClient.c` with it.

It is very important to not include outliers in this list. After running
the test a couple times you'll likely run into a range that sticks
out from the rest of the ranges you already have.

You will need to find offsets for your device/version to run this test.

First, to find `kernel_memory_allocate`, simply xref
`kernel_memory_allocate: VM is not ready`. When you have the offset
set `kma`'s value to it inside `install_kernel_memory_alloc_hook`.

Second, to isolate the test from the other allocations XNU makes,
I test for a specific return address. That address is inside
`OSData::initWithCapacity`. You can easily find OSData's vtable
by xrefing the string `"OSData"`. The first xref to that string
will be in a function that has an xref to the vtable for OSData::MetaClass.
Right above that vtable is OSData's vtable, and `OSData::initWithCapacity`
is at `+0x78`.

Once you have `OSData::initWithCapacity`, find the only `BL` to
`kernel_memory_allocate` and take the offset of the instruction *right below
it*. Inside `kernel_hooks.c`, use that offset in the only if statement
in the only function in that file.

A12+ will need to use something like Correlium.
