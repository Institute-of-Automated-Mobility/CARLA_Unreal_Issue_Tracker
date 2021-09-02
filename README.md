# CARLA_Unreal_Issue_Tracker
### This page is about documenting any issue encountered while working on CARLA or Unreal Engine

## INTEL-MESA: error

#### OS: Ubuntu 20.04
#### Unreal Engine Version: 4.26
#### Graphics: Mesa Intel® UHD Graphics P630 (CFL GT2)

After successful installation of Unreal Engine, upon launching `UE4Editor` if you encounter the following error:
```
INTEL-MESA: error: …/src/intel/vulkan/anv_device.c:2760: GPU hung on one of our command buffers (VK_ERROR_DEVICE_LOST)
[2020.03.23-17.03.13:389] 2]LogVulkanRHI: Error: VulkanRHI::vkQueueSubmit(Queue, 1, &SubmitInfo, Fence->GetHandle()) failed, VkResult=-4
```
and your Unreal Engine Editor's splash screen hung up, then following is the fix:

```{bash}
sudo su
echo "5000" > /sys/class/drm/card0/engine/rcs0/preempt_timeout_ms
```
Note that above fix doesn't persist upon reboot, so you will have repeat every time you reboot.
Once you have complete above fix, you can open Unreal Editor as follows, provided your Unreal Engine repos is in the home directory:

```
cd   ~/UnrealEngine_4.26/Engine/Binaries/Linux
./UE4Editor
```

## Solving the need to use clang-8 while doing `make PythonAPI` for carla

1. Ubuntu 20.04 has clang-9 instead of clang-8. However, installation scripts asks for clang-8. There is nothing wrong with using clang-9. I replaced clang-8 with clang-9 in installation scripts as per https://web.archive.org/web/20210902060002/https://www.simonwenkel.com/2019/04/17/Installing-CARLA-on-Linux-with-current-llvm-clang.html. Of course, this link describes procedure for replacing clang-7 with clang-8 but the idea is same
2. Similarly, you need to replace `llvm-8` with `llvm-9`.

In spite of that, while doing `make PythonAPI`, you may encounter following error:

```
../../LibCarla/source/test/common/test_streaming.cpp:58:3: error: reference to 'Server' is ambiguous
Servertcp::Server srv(io.service, TESTING_PORT);
```

Following is an explanation for this error and how to fix this:


Apparently there is an ambiguity issue in file `LibCarla/source/test/common/test_streaming.cpp` due to
1. the declaration of class Server in both `carla/streaming/Server.h` and `carla/streaming/low_level/Server.h`
2. the declaration of class Server in both `carla/streaming/Client.h` and `carla/streaming/low_level/Client.h`

Solved with a few modifications :
1. Line 58 and 92, replace `Server` by `carla::streaming::low_level::Server`
2. Line 63 and 95, replace `Client` by `carla::streaming::low_level::Client`

You can check the diff of changes  that was made to fix this here: https://gist.github.com/rahulbhadani/f5b18ddaad0ef0daba10d9b57c0499d3

