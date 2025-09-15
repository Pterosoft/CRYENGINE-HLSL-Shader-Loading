# CRYENGINE HLSL Shader Loading

This project implements a runtime-loadable HLSL fullscreen shader system for DirectX 12, designed to apply custom post‑processing effects to the rendered scene. It allows developers and artists to create, modify, and hot‑reload .hlsl pixel shaders without rebuilding the entire application, making it ideal for rapid iteration of visual effects.



At its core, the system:



* Loads and compiles HLSL shaders directly from disk at runtime.
* Integrates seamlessly into the rendering pipeline as a fullscreen pass.
* Provides access to key scene textures (color, depth, normals) and a constant buffer for effect parameters.
* Supports automatic recompilation when shader files are modified.
* Outputs directly to a chosen render target, enabling flexible chaining of effects.



The included example shader, Comics.hlsl, demonstrates a stylized “comic book” rendering effect, combining edge detection, posterization, hatching, and tinting to transform the scene into a hand‑drawn aesthetic. By following the provided guidelines, you can easily create new shaders that plug into the same system — whether for stylization, color grading, distortion, or other creative post‑processing.



## High-Level Overview



Your setup is a runtime HLSL fullscreen pass system for DirectX 12 inside CryEngine’s rendering pipeline. It lets you:



Load a .hlsl file from disk (absolute or relative path).



Compile it at runtime with DXC (DirectX Shader Compiler).



Bind it as a pixel shader with a built-in fullscreen triangle vertex shader.



Feed it scene textures (color, depth, normals) and a constant buffer with effect parameters.



Render the effect directly into a target texture (usually the backbuffer or an intermediate render target).



## Key C++ Components



###### ***CFullscreenHlslPass class***



This is your main controller for loading, compiling, and executing the shader.



###### ***FunctionPurposeUpdateSettings(bool enabled, const char\* fileOrName)***



Enables/disables the pass, sets the shader file path, resolves it, and triggers compilation.



###### ***EnsureUpToDate()***



Checks if the .hlsl file has changed on disk and recompiles if needed.



###### ***CompileAndBuildPipeline()***



Reads the .hlsl file, detects the entry point (//@entry tag or defaults to ExecutePS), compiles the pixel shader, and compiles the fullscreen vertex shader if not already done.



###### ***CompileVSInline()***



Compiles a minimal fullscreen triangle vertex shader directly from a string.



###### ***BuildRawPipelineForFormat(DXGI\_FORMAT)***



Creates the root signature and pipeline state object (PSO) for the given render target format.



###### ***SetResources(CTexture color, CTexture depth, CTexture normals)***



Sets the scene textures to be bound as SRVs.



###### ***UpdateParams(const SComicsParams\&)***



Updates the constant buffer with effect parameters.



###### ***EnsureDescriptors()***



Allocates and updates descriptor heaps for CBV/SRV/Sampler and RTV.



###### ***ExecuteRaw(CTexture target)***



Binds the PSO, root signature, descriptor heaps, and draws the fullscreen triangle.



###### ***Execute()***



Public entry point — checks state, updates parameters, and calls ExecuteRaw.



## How It Fits in the Graphics Pipeline



The execution flow in a frame looks like this:



Setup phase (once or when settings change):



UpdateSettings(true, "Comics.hlsl") → loads and compiles shader.



SetResources(sceneColor, sceneDepth, sceneNormals) → binds textures.



UpdateParams(params) → sets effect parameters.



Per-frame draw:



Execute() is called from the post-processing stage.



It ensures the shader is up-to-date (EnsureUpToDate()).



It ensures descriptors and PSO are ready (EnsureDescriptors() + BuildRawPipelineForFormat()).



It binds everything and issues a DrawInstanced(3, 1, 0, 0) — a fullscreen triangle.



Shader execution:



The fullscreen vertex shader generates UVs for the pixel shader.



The pixel shader (ExecutePS in Comics.hlsl) reads scene textures, applies posterization, outlines, hatching, tint, and outputs the final color.



## The Shader (Comics.hlsl)



This is a stylized “comics” effect with:



Edge detection (depth + luma-based outlines).



Posterization (reducing color levels).



Hatching (crosshatch shading in darker areas).



Tinting (color grading).



Sky/sun handling (avoids artifacts in bright skies).



Inputs:



SceneColor (t0) — main scene color buffer.



SceneDepth (t1) — depth buffer.



SVOGIIndirect (t2) — optional GI buffer.



Constant buffer ComicsCB — effect parameters.



Main function:



ExecutePS(VSOut IN) — applies the full effect and returns the final pixel color.



## Calling It in the Pipeline



From the engine’s perspective, you’d typically:



***auto\& pass = CFullscreenHlslPass::Get();*** 



***pass.UpdateSettings(true, "Comics.hlsl");*** 



***pass.SetResources(sceneColorTex, sceneDepthTex, sceneNormalsTex);***



***pass.UpdateParams(myParams); pass.SetExplicitTarget(outputTex); pass.Execute();***



This would run your shader as a fullscreen pass into outputTex.



## Creating a New Shader (Guidelines)



If you want to make a new fullscreen effect:



Copy the HLSL file (e.g., Comics.hlsl → MyEffect.hlsl).



Change the entry point name if needed:



Add //@entry MyPixelShader at the top.



Implement float4 MyPixelShader(VSOut IN) : SV\_Target0.



Keep the same resource bindings unless you also change the C++:



t0 = SceneColor, t1 = SceneDepth, t2 = optional extra texture.



b0 = constant buffer for parameters.



Adjust parameters:



Update the SComicsParams struct in C++ if you add/remove parameters.



Match the layout in your HLSL cbuffer.



Compile and test:



Call UpdateSettings(true, "MyEffect.hlsl") in code.



Pass in your parameters via UpdateParams().



Optional: Add new samplers or textures — but remember to update:



Root signature creation in BuildRawPipelineForFormat().



Descriptor heap sizes in EnsureDescriptors().



## Pro Tips



Hot reload: The system already detects file changes and recompiles automatically.



Performance: Keep your pixel shader simple — it runs for every pixel on screen.



Debugging: Use -Zi -Od in debug builds for shader debugging.



Formats: The RTV format is auto-detected from the output texture, but if you need HDR, ensure your shader output matches the format.





## We need your help!



We are small indie game studio with no funding and it is becoming hard for us to sustain our development.



If you can [please consider supporting us on Patreon](www.patreon.com/c/pterosoftstudio).



Thank you!

