<h1 align="center">
 ⛯ Divine Shaders ⛯
</h1>

<p align="center">
<img src="https://divine-star-software.github.io/DigitalAssets/images/logo-small.png">
</p>

---

A tool for building GLSL and in the future WGLS shaders.

It allows you to define functions, snippets, varying, uniforms, attributes, and more. 

Snippets are re-usable bits of code that can be called in the shader.

Example:
```ts
import { DivineShaderBuilder as builder } from "divine-shaders";
 builder.snippets.create({
  id: "standard_position",
  body: {
   GLSL: () => `
    vec4 worldPosition = world * vec4(position, 1.0);
    gl_Position = viewProjection * world * vec4(position, 1.0); 
    `,
  },
 });
shader.setCodeBody("vertex", `@standard_position`);
```



Here is a basic exmaple of building and compiling a shader:
```ts

 builder.functions.create("doFog", {
  setID: "#do_fog",
  inputs: [["base", "vec4"]],
  output: "vec3",
  arguments: {},
  body: {
   GLSL: () => `
   if(fogOptions.x == 0.) {
    float fog = ExponentialFog();
   return fog * base.rgb + (1.0 - fog) * vFogColor;
   }
   if(fogOptions.x == 1.) {
     float fogFactor = VolumetricFog();
     return mix( base.rgb, vFogColor, fogFactor );
   }
   if(fogOptions.x == 2.) {
 float fogFactor = AnimatedVolumetricFog();
   return mix( base.rgb, vFogColor, fogFactor );
   }
   return base.rgb;`,
  },
 });


  const shader = DivineShaderBuilder.shaders.create(id);
  shader.addAttributes([
   ["position", "vec3"],
   ["normal", "vec3"],
  ]);
  shader.loadInFunctions(["#do_fog]);
  shader.addUniform(
   [
    ["fogOptions", "vec4"],
    ["vFogInfos", "vec4"],
    ["vFogColor", "vec3"],
    ["time", "float"],
    ["cameraPosition", "vec3"],
    ["cameraDirection", "vec3"],
   ],
   "shared"
  );
  shader.addVarying([
   {
    id: "cameraPOS",
    type: "vec3",
    body: {
     GLSL: () => "cameraPOS = cameraPosition;\n",
    },
   },
   {
    id: "worldPOS",
    type: "vec3",
    body: {
     GLSL: () => `vec4 worldPOSTemp =  world * vec4(position, 1.0);
worldPOS = vec3(worldPOSTemp.x,worldPOSTemp.y,worldPOSTemp.z);`,
    },
   },
   {
    id: "vDistance",
    type: "float",
    body: {
     GLSL: () => " vDistance = distance(cameraPOS , worldPOS );\n",
    },
   },
  ]);
  shader.addUniform(
   [
    ["world", "mat4"],
    ["viewProjection", "mat4"],
   ],
   "vertex"
  );
  shader.setCodeBody("vertex", `@standard_position`);
  shader.setCodeBody(
   "frag",
   `vec3 c = vFogColor.rgb;
c.r -= .2;
c.g -= .2;
c.b -= .2;
vec4 skyboxColor = vec4(c.rgb,1);
vec3 finalColor = doFog(skyboxColor);
gl_FragColor = vec4(finalColor.rgb,1);`
  );
  console.log(
   shader.compiled.vertex,
   shader.compiled.fragment
  )
```