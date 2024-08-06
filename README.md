# GTX-RTEMU-bin-patch 5.4.3

Click the Green button about to get a zipped format and restore to  Engine\Binaries\Win64\UnrealEditor-D3D12RHI.dll overwrite

Solved : Cant Enable RayTracing in UE5 (gtx 1060)(Nvidia Pascal)

at [Epic Forum page](https://forums.unrealengine.com/t/solved-cant-enable-raytracing-in-ue5-gtx-1060-nvidia-pascal/231479/127)

The first change was to add the ability to use the variable DXR_ALLOW_EMULATED_RAYTRACING in the WindowsD3D12Device.cpp file.
I’m not making games and just using the path tracer to make images, so i set this to be on by default so i wouldn’t have to edit anything in the ConsoleVariables.ini file. After compiling it (thankfully it took only a few seconds to compile) Unreal ran, but i wasn’t getting any ray tracing. Checking the Output Log, and searching for ‘D3d12’ i saw :

Ray tracing is disabled because D3D12 ray tracing tier 1.1 is required but only tier 1.0 is supported
Which seems to have moved the tier 1.0 cards to the ‘do not enable’ side of the logic.
Undoing this change got me up and running with ray tracing.

This worked for me and my 1080. Hope it will help others.

Here are the files with their changes:
…\UnrealEngine\Engine\Source\Runtime\D3D12RHI\Private\Windows\WindowsD3D12Device.cpp

change
```
#define DXR_ALLOW_EMULATED_RAYTRACING 0

#if DXR_ALLOW_EMULATED_RAYTRACING
int32 GAllowEmulatedRayTracing = 0;
to

#define DXR_ALLOW_EMULATED_RAYTRACING 1

#if DXR_ALLOW_EMULATED_RAYTRACING
int32 GAllowEmulatedRayTracing = 1;
```
and

…\UnrealEngine\Engine\Source\Runtime\D3D12RHI\Private\D3D12Adapter.cpp

change
```
UE_LOG(LogD3D12RHI, Log, TEXT("Ray tracing is disabled because D3D12 ray tracing tier 1.1 is required but only tier 1.0 is supported."));
```
to
```
UE_LOG(LogD3D12RHI, Log, TEXT("Ray tracing emulation is now enabled. Use at own risk. D3D12 tier 1.1 check ignored. Tra-La-La!"));
GRHISupportsRayTracing = RHISupportsRayTracing(GMaxRHIShaderPlatform);
GRHISupportsRayTracingShaders = GRHISupportsRayTracing && RHISupportsRayTracingShaders(GMaxRHIShaderPlatform);
```
