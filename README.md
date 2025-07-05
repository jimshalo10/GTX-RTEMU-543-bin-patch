# GTX-RTEMU-bin-patch 5.4.3 and 5.6 

Update:- The instructions have been rewritten to allow easier updating as new UE5 versions

**The previous method dumped lots of the files in the wrong places and easily placed the wrong file for the wrong version.**


This repository now contains ALL the versions made for UE 5. The version for 5.5 was corrupted and deleted

Click the Green button above, scroll to the bottom, and select "Download Zip" to obtain a zipped file format.

Restore by unzipping to Downloads\GTX-RTEMU-543-bin-patch, follow the instructions for your version

**5.4**

C:\Program Files\Epic Games\UE_5.4

In the Windows File Explorer, copy the version for 5.4 

If you use the Windows Command Shell, open it with "Run As Administrator" and copy and paste using the icon in the top right for both commands

```
copy Downloads\GTX-RTEMU-543-bin-patch\ue54\UnrealEditor-D3D12RHI.dll  C:\Program Files\Epic Games\UE_5.4\Engine\Binaries\Win64\UnrealEditor-D3D12RHI.dll 
```

The patched control file is in this package 

```
copy Downloads\GTX-RTEMU-543-bin-patch\Engine\Config\ConsoleVariables.ini  C:\Program Files\Epic Games\UE_5.4\Engine\Config\ConsoleVariables.ini 
```


Make sure you have changed the control files as described in the Epic Forum Thread correctly

Solved : Cant Enable RayTracing in UE5 (gtx 1060)(Nvidia Pascal)

at [Epic Forum page](https://forums.unrealengine.com/t/solved-cant-enable-raytracing-in-ue5-gtx-1060-nvidia-pascal/231479/127)

**Update for 5.6**

**5.6**

In the Windows File Explorer, copy the version for 5.6 

If you use the Windows Command Shell, open it with "Run As Administrator" and copy and paste using the icon in the top right for both commands

```
copy Downloads\GTX-RTEMU-543-bin-patch\ue56\UnrealEditor-D3D12RHI.dll  C:\Program Files\Epic Games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-D3D12RHI.dll 
```

The patched control file is in this package 

```
copy Downloads\GTX-RTEMU-543-bin-patch\Engine\Config\ConsoleVariables.ini  C:\Program Files\Epic Games\UE_5.6\Engine\Config\ConsoleVariables.ini 
```



**LEGAL STUFF
If you use this patch, there is NO WARRANTY, NO SUPPORT and NO LIABILITY BY EPIC AND ANY OTHER PARTIES
Your installation of 5.4 or 5.6 binary will not be production**

_Changes made to make 5.6 work_

The first change was to add the ability to use the variable DXR_ALLOW_EMULATED_RAYTRACING in the WindowsD3D12Device.cpp file.
I’m not making games and just using the path tracer to make images, so i set this to be on by default so i wouldn’t have to edit anything in the ConsoleVariables.ini file. After compiling it (thankfully, it took only a few seconds to compile) Unreal ran, but i wasn’t getting any ray tracing. Checking the Output Log, and searching for ‘D3d12’ i saw :

Ray tracing is disabled because D3D12 ray tracing tier 1.1 is required, but only tier 1.0 is supported
Which seems to have moved the tier 1.0 cards to the ‘do not enable’ side of the logic.
Undoing this change got me up and running with ray tracing.


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

For version 5.5.1 the followng patch is needed as there will be a UE5 crash
```
Thanks. I’ve tested it and it doesn’t work for me at all. UE crashes with error saying:

Assertion failed: GRHIGlobals.RayTracing.SupportsDispatchIndirect [File:C:\Users\Owner\source\repos\ue5Lgpu551\Engine\Source\Runtime\D3D12RHI\Private\D3D12RayTracing.cpp] [Line: 4977] 
RHIRayTraceDispatchIndirect may not be used because DXR 1.1 is not supported on this machine.
```

The fix to this is to comment out the failing line to stop checking of Tier 1.1 

…\UnrealEngine\Engine\Source\Runtime\D3D12RHI\Private\D3D12RayTracing.cpp at about line 4977

```
	//@GTXEMU Remove teir 1.1 crash
	//checkf(GRHISupportsRayTracingDispatchIndirect, TEXT("RHIRayTraceDispatchIndirect may not be used because DXR 1.1 is not supported on this machine."));
```

