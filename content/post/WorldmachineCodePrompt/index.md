---
title: "WorldmachineCodePrompt"
date: 2025-07-03T13:09:10+08:00
draft: false
description: "使用AI辅助编写Worldmachine中Code模块代码的提示词。"
image : ""
categories:
- 技术分享
tags:
- 地形
- Worldmachine
---
# **角色与目标 (Role and Goal)**

你是一位精通 World Machine Code Device 的专家级程序员。你的任务是基于我提供的官方文档、代码示例和具体需求，为我编写一个全新的、功能完整的自定义节点。你需要提供该节点所需的 **Host (Lua) 代码**和 **Compute (OpenCL) 代码**，以及其使用方法。当程序不通时，你需要根据我补充的报错进行修复。

# **第一部分：背景知识**

在开始之前，请先学习以下关于 World Machine Code Device 的核心文档。

## Code Device

This feature was introduced in the “[Hurricane Ridge](https://help.world-machine.com/lookup/hurricaneridge/)” release

The Code device lets you write custom devices for World Machine. These device plugins use GPU compute and can be shared with other users safely, due to the sandboxed execution model.

## Overview

Your Code device contains custom programming that you create as follows:

1. A script controlling the execution of the device, written in [Lua](https://www.lua.org/).
2. Zero or more compute kernels for executing code on the GPU.

When you create a new Code device and open it for editing, you will see the Code Editor view:

![img](https://help.world-machine.com/wp-content/uploads/2024/11/image.png)

- The tabs across the top allow you to access the **Host code** (scripting language) and the **Compute code** (OpenCL or equivelent).
- The tabs at the bottom show you any reported errors during compilation or execution of your device in the first tab, and any host script output (such as that created by the print() function) in the second tab.
- The right side drawers allow you to modify the configuration of your custom device, including adding ports, parameters, and more.
- The “apply changes” button commits any changes you’ve written in your code and rebuilds the device.

## Languages

Your device is written in both a host language and a compute language.

- The host language is a slow scripting language executed on the CPU. Your host script acts to orchestrate the execution of your device, but should perform no major calculations itself.
- The compute language is a fast, data-parallel language executed on the GPU. This is used for code that manipulates large amounts of data, such as the entire contents of a heightfield or image.

### Host Language

Currently **Lua** is the allowed host script language. Other languages may be supported in the future.

The Lua environment is by default heavily sandboxed. Other than core language function, only the standard math library is enabled. Your script will interact with World Machine via the **World Machine library** (exposed as ‘wm’).

This looks much like:

```
-- Create an empty memory buffer
wm.createBuffer()

-- Get a parameter
local myFloat = wm.parm("parm0")

-- Output to port 0
wm.output(bufResult, 0)
```

See the [World Machine Lua API Reference ](https://help.world-machine.com/topic/plugin-device-lua-api/)for complete details of how to interact with World Machine.

### Compute Language

Currently **OpenCL** is the compute language. Other languages may be supported in the future. OpenCL 1.2 is the minimum version required. Compute kernels are compiled by your GPU drivers and run on the GPU.

Any valid compute kernel can be run within World Machine.

See the Language References section for links for general information on writing compute kernels in OpenCL C.

## Configuration

A number of configuration options are available for your device:

![img](https://help.world-machine.com/wp-content/uploads/2024/11/image-1.png)

- **Identity** is the most important thing to assign. When you click the **Assign** button, you can name your device. **Naming the device embeds a hidden globally unique class identifier.** Devices that have the same GUID will be considered the “same” in the eyes of World Machine, in much the same way that different Basic Perlin devices are the same. This enables you to copy/paste settings between those devices, etc. If you change the Identity of the device, it will no longer be considered the same… do so carefully.
- The Family of the device controls its appearance, and where it gets sorted within the various menus and toolbars in World Machine
- The Summary should be a brief description of what the device does
- The Description can be more in depth. It is displayed when the user clicks a help icon.

Parameters, inputs, and outputs are created the same as for Macros. To retrieve your parameters from the script, refer to it by the ID you’ve given it – this can be the automatically assigned one, or a custom one. For example:

![img](https://help.world-machine.com/wp-content/uploads/2024/11/image-2.png)

## Limitations

There are a number of limitations right now in the Code device.

- The scripting bindings to World Machine are still **under construction**.
- Currently only supports heightfield and bitmap datatypes for compute.
- You can only provide a build script. No other control of the device is possible.
- There is no access to built-in functions, such as rescaling a packet. This would be very useful.
- Only Lua and OpenCL are supported

Some of these limitations will be improved over time.

## Resources

### World Machine

- [World Machine Lua API](https://help.world-machine.com/topic/plugin-device-lua-api/)

### Language References

- [OpenCL API Reference Sheet](https://www.khronos.org/files/opencl30-reference-guide.pdf)
- [OpenCL Kernel Language Reference](https://registry.khronos.org/OpenCL/specs/3.0-unified/html/OpenCL_C.html)

- [Lua – Getting Started](https://www.lua.org/pil/contents.html)
- [Lua Reference Guide](https://www.lua.org/manual/5.4/)

### CPU Drivers

The default GPU backend is **OpenCL**. Although the Code device is intended to be GPU accelerated, you may wish to use the Intel OpenCL CPU driver to run code on your processor.

- [Intel OpenCL CPU Drivers](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-cpu-runtime-for-opencl-applications-with-sycl-support.html)

The Intel driver has been tested and works well. (Yes, you can also use it with AMD-based systems).

For many common scenarios within World Machine, the OpenCL performance running on the CPU is competitive with the GPU.

以及LUA Plugin API文档。

## **LUA World Machine Plugin API**

This feature was introduced in the “[Hurricane Ridge](https://help.world-machine.com/lookup/hurricaneridge/)” release

This article contains the currently supported API for interacting with World Machine from your Lua scripts within the Code device.

As the bindings are continually developed, this reference material will be updated to stay current.

## Overview

This API reference is only for the **Host script language**. You will also need to write one or more kernels in your compute language to create a device.

All World Machine bindings are exposed in the ‘wm’ library. To use any of the top-level commands below, make sure you access them with the ‘wm’ prefix. For example, ‘wm.parameter()’.

## Sandboxing

To allow for secure sharing of plugin devices between users, the Lua scripting session is sandboxed:

- Your script has no access to the system or external data.
- Your script cannot persist information outside of the device build process.
- The compute kernels have no access to any other data than that provided via the scripting.
- The lua math and string libraries are available. No other built-ins are provided.
- The World Machine plugin API *is* accessible.

Future versions may relax the sandboxing for custom devices that are not shared with others.

## Datatypes

A variety of custom datatypes are created by the World Machine library.

**buffer**: A raw collection of bytes. Can be used to supply general memory to your compute kernel, or when accessing the underlying data of an input such as a heightfield. Your kernel should accept this as a **float\*** parameter (or other relevant type).

**image**: An Image datatype, as defined in OpenCL. This may be more convenient than accessing a raw buffer, and some API functions will only return images. The image will be a 2D four channel RGBA texture. Your kernel must access as an **image2d_t** parameter.

In addition, standard Lua datatypes are used:

**value**: This is a Lua built-in integer or floating point value. Your kernel can accept either and WM will convert as appropriate.

**table**: A Lua table, which is simply a collection of values.

**string**: A Lua string.

A few additional notes:

- If a function takes a slot value, it should be provided an integer.
- If a function takes a name, it should be provided a string.
- The extended datatypes have reference semantics. For example, that means that assigning one buffer to a new buffer variable does not copy the buffer – they will both refer to the same object.

### Definitions

Some values defined within the library are meant to be identifiers rather than be manipulated. Providing these values allows WM to know what kind of data you are referring to.

#### Packet types

You can pass these identifiers to createBuffer or createImage, among other places, to create the correct types. They are a table defined at ‘wm.packet’.

```
wm.packet.hf -- A single channel heightfield image

wm.packet.rgb -- A 3-channel rgb image.
```

#### Data types

These are values used to communicate how to interpret a raw buffer. They are a table defined at ‘wm.type’. These are unused in the current API.

```
wm.type.byte -- 8bit unsigned integer data

wm.type.int -- 32bit

wm.type.int64 -- 64bit

wm.type.float -- 32bit floating point

wm.type.vec2 -- float vector2

wm.type.vec3 -- float vector3

wm.type.vec4 -- float vector4

wm.type.half -- 16bit floating point

wm.type.half3 -- 3 x 16bit floats
```

## Inputs

Functions to take data from the input ports on the device, as well as the current parameters. The slot should be a number starting at zero for the first input.

```
buffer wm.inputBuffer(slot)

image wm.inputImage(slot)

value wm.inputValue(slot)

image wm.mask()

value wm.parameter(name)

value wm.parm(name) -- A convenience short alias for the above
```

Inputs of either heightfield or bitmap type can be taken as images or buffers, using either of the first two functions above as needed. The returned result will be of the type shown and can be used or passed to other functions.

Note that if you take a bitmap input as a buffer, the data is 3 channels of 16bit floating point, which takes special consideration to access inside of an OpenCL compute kernel.

## Creation

Create new memory buffers or images for use by your compute kernels.

- If no type is specified, the default is a heightfield.
- If no dimensions are specified, the current world render size is used.

**The dims parameter is currently ignored!** It is not currently possible to create images of a different size than the general context.

```
image wm.createImage(type, dims)

buffer wm.createBuffer(type, dims)

buffer wm.createRawBuffer(bytes)
```

## Context Functions

Several functions are built-in to allow you to gain access to the build context your device is invoked within.

### Worldspace

```
image wm.worldspace()
```

Generate a image containing the the xy world-space coordinates for this build, in the red and green channels. The blue and alpha channels are unused and set to zero.

The image values are unnormalized 32bit floats. They correspond exactly to the render extents defined in your project settings.

Note that the worldspace coordinates are in internal World Machine units instead of kilometers. Each unit corresponds to 8 kilometers.

This is **very useful** to allow your compute kernel to understand what point in the world each pixel corresponds to! You will often use this, or the devicespace() function, to easily access this information.

If you output this image directly as a bitmap, it would look like this:

![img](https://help.world-machine.com/wp-content/uploads/2024/11/image-4.png)

### Devicespace

```
image wm.devicespace()
```

Devicespace is the world-space coordinate transformed by the device’s placement. The placement includes rotation, translation, and any connected domain distortion input.

The RGBA image is divided into two pairs of channels. The image values are unnormalized 32bit floats with real numbers.

- The RG channels contains the (X,Y) distorted coordinates
- The BA channels contains the undistorted (X,Y) device coordinates.

It looks like this, visualized:

![img](https://help.world-machine.com/wp-content/uploads/2024/11/image-3.png)

Generally you will only need the RG channels. Device space is very useful for generator-type devices that want to make a pattern or some other output that can be placed into the world.

### World Context

Retrieve a table containing the current render extent information.

```
table wm.context()
```

The table has the following members:

int dimX, dimY — Dimensions in pixels of the render extent

float x0, y0, x1, y1 — World render extent corners

float originX, originY — Placement origin

float rotate2D — Placement rotation in degrees

## Outputs

Send data to an output port.

```
wm.output(data, port)
```

The port should be a number, starting at zero for the first output.

The port configuration determines the acceptable datatype to send. For example, a heightfield output must contain a buffer or image. Possible types of your data include:

**hf** : image, buffer

**rgb** : image, buffer

**text** : string

**param** : value, string

## Compute Kernels

To perform operations on the GPU, use the runKernel function in the wm library.

```
wm.runKernel(kernelname/kernelobject, arg1, arg2, ...)
```

The first parameter must be a string containing the name of the compute kernel, while the following arguments should be buffers, images, or values. The types must match the type defined in your kernel function.

Currently, there is no way to define the local workgroup size of a kernel.

## Other Functions

### Progress Reporting

```
wm.progress(percent)

wm.progress(current, total)
```

You can specify the build progress as a floating point value from 0..1 or as current, total as integers.

There is currently no way to check if the user has requested the build to cancel.

# **第二部分：学习示例**

这里有几个完整的代码示例，请学习它们的结构和实现方式。每个示例都包含 **Host (Lua)** 和 **Compute (OpenCL)** 两部分。

## 1.OpenCL Example Generator

Host Part

```
-- The featurescale is a distance value that will correspond to the wavelength of a full sinewave, in this example
local featurescale = wm.parameter("featureScale")

-- Switch the kernel we're using based on the option chosen by the user
local useKernel = "primary"
local kerneltype = wm.parm("type")
if (kerneltype == 1) then
    useKernel = "secondary"
end


-- Get the per-pixel devicespace location. This automatically accounts for device origin, rotation, and distortion
wspace = wm.devicespace()

-- Create a buffer for the compute result. By default, it will be a heightfield (sngle channel 32bit float) at the current device build size
result = wm.createBuffer()

-- Run the compute kernel!
wm.runKernel(useKernel, wspace, result, featurescale)

-- Transfer the results and output from this device
wm.output(result, 0)
```

Compute Part

```
void kernel primary(__read_only image2d_t location, global float* A, float featureScale) {
    int2 gridPos = (int2)(get_global_id(0), get_global_id(1));
    size_t stride = get_global_size(0);
    size_t idx = stride * gridPos.y + gridPos.x;
    
    float4 xy = read_imagef(location, gridPos);
    
    float freq = 2.0f * 3.14159f / featureScale;
    float result = sin(xy.x * freq); // along X axis

    A[idx] = result * 0.5f + 0.5f;
};

void kernel secondary(__read_only image2d_t location, global float* A, float featureScale) {
    int2 gridPos = (int2)(get_global_id(0), get_global_id(1));
    size_t stride = get_global_size(0);
    size_t idx = stride * gridPos.y + gridPos.x;
    
    // Get the xy location from our input image. Adjust the center of the radial gradient to the middle of our default extents (0.5, 0.5)
    //  as a positioning offset example
    float2 xy = read_imagef(location, gridPos).xy - (float2)(0.5f, 0.5f);

    float freq = 2.0f * 3.14159f / featureScale;
    float result = cos(length(xy) * freq);

    A[idx] = result * 0.5f + 0.5f;
};
```

## 2.OpenCL Blur

Host Part

```
-- This is a simple example of a blur kernel, applied 'Repeat' number of times to the image.
-- The blur kernel performs a simple 3x3 box filter convolution.

map = wm.inputBuffer(0)
temp = wm.createBuffer()

maxIter = wm.parameter("Repeat")

-- Do ping-pong buffering. Each iteration blurs twice, so that the results end up in the same buffer we started with

for i = 1, maxIter do
    wm.runKernel("diffuse", map, temp)
    wm.runKernel("diffuse", temp, map)
    wm.progress(i, maxIter)
end
wm.output(map, 0)
```

Compute Part

```
extern size_t gridIndex();

void kernel diffuse(global float* A, global float *B) {
    size_t idx = gridIndex();

    int2 sz = (int2)(get_global_size(0), get_global_size(1));
    int2 coord = (int2)(get_global_id(0), get_global_id(1));
    
    bool oob = false;
    
    if (coord.x < 1 || coord.y < 1)
        oob = true;
       
    if (coord.x >= sz.x-1 || coord.y >= sz.y-1)
        oob = true;

    // Simple out of bounds protection by not blurring the edge. This leaves a one pixel unblurred border which is undesirable but works fine for this example.        
    if (oob) {
        B[idx] = A[idx];
        return;
    }

    size_t stride = get_global_size(0);


    float r = A[idx] + A[idx-1] + A[idx+1];   
    r += A[idx-stride] + A[idx+stride];
    r += A[idx-stride-1] + A[idx+stride-1];
    r += A[idx-stride+1] + A[idx+stride+1];
    
    
    r /= 9;

    B[idx] = r;
};
```

## 3.OpenCL FlowDirs

Host Part

```
bufInput = wm.inputBuffer(0)
bufResult = wm.createBuffer()

wm.runKernel("flowdir", bufInput, bufResult)
wm.output(bufResult, 0)
```

Compute Part

```
//Compute kernel
void kernel flowdir(global float* A, global float *result) {

    float r = 0.0f;
    
    int x = get_global_id(0);
    int y = get_global_id(1);

    size_t width = get_global_size(0);
    size_t height = get_global_size(1);
    size_t stride = width;
    size_t idx = stride * get_global_id(1) + get_global_id(0);
    
    // Have to protect against out of buffer access
    if (y > 0 &&  y < height-1) {
        if (x > 0 && x < width-1) {
            float cur = A[idx];
            float w = A[idx-1] - cur;
            float e = A[idx+1] - cur;
            float n = A[idx+stride] - cur;
            float s = A[idx-stride] - cur;
            
        
            float best = 0.0f;
            if (w >  best) {
                r = 0.2f;
                best = w;
            }
            if (e >  best) {
                r = 0.4f;
                best = w;
            }
            if (n >  best) {
                r = 0.6f;
                best = w;
            }
            if (s >  best) {
                r = 0.8f;
                best = w;
            }
        }                    
    }           
    
    result[idx] = r;
};
```

## 4.OpenCL PerlinNoise

Host Part

```
-- This is an example showing how to create a complex procedural generator such as Perlin noise.
-- The overall process is still simple.

wspace = wm.devicespace()
result = wm.createBuffer()

local featureScale = 1.0 / wm.parameter("parm0")
local amplitude = wm.parameter("parm1")
local octaves = 16

wm.runKernel("gpuPerlin", result, wspace, featureScale, amplitude, octaves)
wm.output(result, 0)
```

Compute Part

```
// Permute table for perlin noise
int pRef[] = { 151,160,137,91,90,15,
131,13,201,95,96,53,194,233,7,225,140,36,103,30,69,142,8,99,37,240,21,10,23,
190,6,148,247,120,234,75,0,26,197,62,94,252,219,203,117,35,11,32,57,177,33,
88,237,149,56,87,174,20,125,136,171,168,68,175,74,165,71,134,139,48,27,166,
77,146,158,231,83,111,229,122,60,211,133,230,220,105,92,41,55,46,245,40,244,
102,143,54,65,25,63,161,1,216,80,73,209,76,132,187,208,89,18,169,200,196,
135,130,116,188,159,86,164,100,109,198,173,186,3,64,52,217,226,250,124,123,
5,202,38,147,118,126,255,82,85,212,207,206,59,227,47,16,58,17,182,189,28,42,
223,183,170,213,119,248,152,2,44,154,163,70,221,153,101,155,167,43,172,9,
129,22,39,253,19,98,108,110,79,113,224,232,178,185,112,104,218,246,97,228,
251,34,242,193,238,210,144,12,191,179,162,241,81,51,145,235,249,14,239,107,
49,192,214,31,181,199,106,157,184,84,204,176,115,121,50,45,127,4,150,254,
138,236,205,93,222,114,67,29,24,72,243,141,128,195,78,66,215,61,156,180, // end of table. 
151,160,137,91,90,15,	// doubled up so can skip clamping
131,13,201,95,96,53,194,233,7,225,140,36,103,30,69,142,8,99,37,240,21,10,23,
190,6,148,247,120,234,75,0,26,197,62,94,252,219,203,117,35,11,32,57,177,33,
88,237,149,56,87,174,20,125,136,171,168,68,175,74,165,71,134,139,48,27,166,
77,146,158,231,83,111,229,122,60,211,133,230,220,105,92,41,55,46,245,40,244,
102,143,54,65,25,63,161,1,216,80,73,209,76,132,187,208,89,18,169,200,196,
135,130,116,188,159,86,164,100,109,198,173,186,3,64,52,217,226,250,124,123,
5,202,38,147,118,126,255,82,85,212,207,206,59,227,47,16,58,17,182,189,28,42,
223,183,170,213,119,248,152,2,44,154,163,70,221,153,101,155,167,43,172,9,
129,22,39,253,19,98,108,110,79,113,224,232,178,185,112,104,218,246,97,228,
251,34,242,193,238,210,144,12,191,179,162,241,81,51,145,235,249,14,239,107,
49,192,214,31,181,199,106,157,184,84,204,176,115,121,50,45,127,4,150,254,
138,236,205,93,222,114,67,29,24,72,243,141,128,195,78,66,215,61,156,180
};

float perlin_fade(float t)
{
    return t * t * t * (t * (t * 6.f - 15.f) + 10.f);
}

float perlin_lerp(float t, float a, float b)
{
    return a + t * (b - a);
}

float perlin_grad(int hash, float x, float y)
{
    float z = 0.0f;
    int h = hash & 15;                      // CONVERT LO 4 BITS OF HASH CODE
    float u = h < 8 ? x : y,                 // INTO 12 GRADIENT DIRECTIONS.
    v = h < 4 ? y : h == 12 || h == 14 ? x : z;
    return ((h & 1) == 0 ? u : -u) + ((h & 2) == 0 ? v : -v);
}

// Reference implementation of perlin noise, lightly modified for opencl
float noise2(float2 loc)
{
    int*p = pRef;
    float2 floc = floor(loc);
    loc -= floc;

    int X = (int)floc.x & 255,                  // FIND UNIT CUBE THAT
    Y = (int)floc.y & 255;                  // CONTAINS POINT.

    float     u = perlin_fade(loc.x),                                // COMPUTE FADE CURVES
              v = perlin_fade(loc.y);                               // FOR EACH OF X,Y

    int A = p[X] + Y, AA = p[A], AB = p[A + 1],      // HASH COORDINATES OF
        B = p[X + 1] + Y, BA = p[B], BB = p[B + 1];      // THE 8 CUBE CORNERS,

    return perlin_lerp(v, perlin_lerp(u, perlin_grad(p[AA], loc.x, loc.y),  // AND ADD
	        perlin_grad(p[BA], loc.x - 1.0f, loc.y)), // BLENDED
	        perlin_lerp(u, perlin_grad(p[AB], loc.x, loc.y - 1.0f),  // RESULTS
	        perlin_grad(p[BB], loc.x - 1.0f, loc.y - 1.0f)));// FROM  4
}

// Same as above, but with the permute table in fast local shared memory
float noise2Fast(float2 loc, local int*p)
{
    float2 floc = floor(loc);
    loc -= floc;

    int X = (int)floc.x & 255,                  // FIND UNIT CUBE THAT
    Y = (int)floc.y & 255;                  // CONTAINS POINT.

    float     u = perlin_fade(loc.x),                                // COMPUTE FADE CURVES
              v = perlin_fade(loc.y);                               // FOR EACH OF X,Y

    int A = p[X] + Y, AA = p[A], AB = p[A + 1],      // HASH COORDINATES OF
        B = p[X + 1] + Y, BA = p[B], BB = p[B + 1];      // THE 8 CUBE CORNERS,

    return perlin_lerp(v, perlin_lerp(u, perlin_grad(p[AA], loc.x, loc.y),  // AND ADD
	        perlin_grad(p[BA], loc.x - 1.0f, loc.y)), // BLENDED
	        perlin_lerp(u, perlin_grad(p[AB], loc.x, loc.y - 1.0f),  // RESULTS
	        perlin_grad(p[BB], loc.x - 1.0f, loc.y - 1.0f)));// FROM  4
}


// Return the index of this cell for global memory arrays
extern size_t gridIndex();

// Return the UV coordinate of the current cell within the render extent
extern float2 uvCoord();

extern int2 gridCoord();

void kernel gpuPerlin(global float* A, image2d_t imgWS, float featureScale, float persist, int octaves) {
    int2 loc = gridCoord();

    float2 uv = read_imagef(imgWS, loc).xy * featureScale;

    float r = 0.0f;
    float amp = 1.0f;
    for (int i =0; i < octaves; i++) {
        r += noise2(uv) * amp;
        uv *= 2.0f;
        amp *= persist;
    }
   
    // store to our buffer    
    int idx = gridIndex();
    A[idx] = 0.5f + 0.5f*r;
};

// Use local shared memory for permute table. On older GPUs, this is faster. On modern devices the table is cached anyways.
void kernel gpuPerlinFast(global float* A, image2d_t imgWS, float featureScale, float persist, int octaves) {

    __local int pL[512];

    // initialize shared memory
    if (get_local_linear_id() == 0) {
        for (int i = 0; i < sizeof(pRef) / sizeof(pRef[0]); i++) {
            pL[i] = pRef[i];                  
        }             
    }

    barrier(CLK_GLOBAL_MEM_FENCE  );


    int2 loc = gridCoord();

    float2 uv = read_imagef(imgWS, loc).xy * featureScale;

    float r = 0.0f;
    float amp = 1.0f;
    for (int i =0; i < octaves; i++) {
        r += noise2Fast(uv, pL) * amp;
        uv *= 2.0f;
        amp *= persist;
    }
   
    // store to our buffer    
    int idx = gridIndex();
    A[idx] = 0.5f + 0.5f*r;
};
```

## 5.OpenCL Simplest

Host Part

```
-- This plugin defines no compute kernel functions of its own - all we do is retrieve the worldspace coordinate and output it!

-- Use the built-in worldspace function to retrieve the worldspace coordinate of every pixel, encoded as an RGB value
mapWorldCoords = wm.worldspace()

-- Output that image with no further modification. Usually, you'd use the image as an input to a kernel, but in our simplest implementation we do nothing with it.
wm.output(mapWorldCoords, 0)
```

Compute Part

```
// This part is intentionally left empty.
```

## 6.Brick Generator

Host Part

```
--- Brick Generator ---
-- Use the built-in worldspace function to retrieve the worldspace coordinate of every pixel, encoded as an RGBA value
mapWorldCoords = wm.devicespace()
mapHeight = wm.createImage(wm.packet.hf)

-- Run an opencl kernel to create the brick texture
wm.runKernel("brick", mapWorldCoords, mapHeight, wm.parm("parm0"), wm.parm("parm1"), wm.parm("parm2"), wm.parm("parm3"), wm.parm("color0"), wm.parm("color1"), wm.parm("colorM"))

-- Output our diffuse image and our heightfield
wm.output(mapWorldCoords, 0)
wm.output(mapHeight, 1)
```

Compute Part

```
extern int2 gridCoord();

// Hash function, used to vary the brick identity
unsigned int hash(unsigned int x)
{
	x ^= x >> 16;
	x *= 0x7feb352dU;
	x ^= x >> 15;
	x *= 0x846ca68bU;
	x ^= x >> 16;
	return x;
}


void kernel brick(__read_write image2d_t imgPos, __write_only image2d_t imgHeight, float scale, float aspect, float mortar, float offsetF, float4 color0, float4 color1, float4 colorM) {

    int2 loc = gridCoord();
    
    float4 v = read_imagef(imgPos, loc);
    
    float2 wpos = v.xy / scale;

    wpos.y *= aspect;

    // We calculate the brick coordinate, and then offset every other row by the offset amount specified    
    bool offset = ((int)floor(wpos.y)) % 2 == 0;

    if (offset)
        wpos.x += offsetF;

    float2 ifpart;
    float2 dist = fract(wpos, &ifpart);
    
    dist = min(dist, 1.0f - dist);

    int2 ipart = (int2)(ifpart.x, ifpart.y);    

    // Create a brick ID code from the brick location. We'll use it to mix colors for pleasing variety       
    int brickCode = ipart.y;
    brickCode = hash(12 + brickCode);
    brickCode = hash(brickCode + ipart.x);
    
    float cellF = (brickCode & 255) / 255.0f;        
    v = mix(color0, color1, cellF );    
    
    // Now mix in the mortar color   
    float b = min(dist.x, dist.y) - mortar;
    if (b < 0) {
        v = colorM;
        b = 0.0f;
    }
    
    // shape heights
    b = sqrt(b);
    
    write_imagef(imgPos, loc, v);        
    write_imagef(imgHeight, loc, b);
}
```

# **第三部分：你的任务**

现在，请根据以上知识和示例，为我创建以下指定的 Code Device。

**节点名称:**风 / Wind

**功能描述:**对来自输入端的 Heightmap 进行类似于 Photoshop 中的风滤镜的处理。

**输入端口:**Input Terrain

**输出端口:**Output Terrain

**可调整参数 (Parameters):**自由发挥。

**代码要求:**

1. **Host (Lua) 代码:** 自由发挥。
2. **Compute (OpenCL) 代码:** 自由发挥。