# UPDATES!

We opened a bug in Unity's bug tracker, and got a reply with a fix. If you reached this page looking for a solution for this problem, you can change `Packages/Universal RP/Shaders/ShadowCasterPass.hlsl` in this way:

```glsl
float4 GetShadowPositionHClip(Attributes input)
{
    float3 positionWS = TransformObjectToWorld(input.positionOS.xyz);
    float3 normalWS = TransformObjectToWorldNormal(input.normalOS);

    float4 positionCS = TransformWorldToHClip(ApplyShadowBias(positionWS, normalWS, _LightDirection));

#if UNITY_REVERSED_Z
    // COMMENT THE LINE BELOW:
    //positionCS.z = min(positionCS.z, positionCS.w * UNITY_NEAR_CLIP_VALUE);
#else
    positionCS.z = max(positionCS.z, positionCS.w * UNITY_NEAR_CLIP_VALUE);
#endif

    return positionCS;
}
```

This change eliminated the issue in URP for us. We still don't know how to fix this in the Built-in pipeline, or if this will be integated in upcoming URP releases. You can follow this thread to get more updates in this subject here:

https://fogbugz.unity3d.com/default.asp?1286234_tah6nrs91m7vbrmk

Thanks to all the gamedev friends and Unity's team who helped us work around this problem!

---

# Context

We're using Unity 2019.4 for this demonstration as it is the current LTS version for Unity. The problem described here also happens in Unity 2020.1 using the same setup.

This problem emerged while we were trying to make a simple prototype of a 3rd person game with a top-down'ish camera, where the controllable character has a flashlight. We decided to go with a realtime spotlight to make this flashlight as it seemed other approaches (baked illumination) would be detrimental to the look we wanted for the game.

# The Setup

For each of the pipelines (URP, HDRP and Built-in), a default project for that pipeline was created from Unity Hub's launcher. All settings for shadow strength, distance, and resolution were the same as the default settings.

On a new scene, we added Unity's default plane and cube 3D objects, a directional light for minimal fill-light, and a spotlight, pointed towards the wall:

![Image of the scene setup](./ReadmeImages/SceneSetup1.png)

# The Problem

When the wall is animated, both in URP and Built-in pipelines, the spotlight is able to go through the middle of the wall.

Built-in pipeline:
The problem happens when using either Deferred or Forward rendering.

![Built-in results](./ReadmeImages/BuiltIn.gif)

URP:

![URP results](./ReadmeImages/URP.gif)

# HDRP does not show the same issue

HDRP does not show the same issue when using either Deferred or Forward rendering.

![HDRP results](./ReadmeImages/HDRP.gif)

HDRP seems to calculate shadowmaps every frame when a light is marked to do so. Either this or the ShadowCaster portion of the HDRP shaders seem to prevent the issue that happens in URP and Built-in, but this is just a suspicion, as it is very difficult to debug this.

# Final Considerations

We're not sure what causes this issue to happen in both Built-in and URP and not happen in HDRP. Though adopting HDRP would fix this issue for us, it makes the person-hour cost of making materials and effects much larger, and prevents us from developing for lower-end PCs and Nintendo Switch (which HDRP doesn't support yet).

Any suggestions for solving or understanding this issue is welcome.