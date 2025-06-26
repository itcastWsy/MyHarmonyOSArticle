# ArkTS 3D Graphics and WebGL Integration

## Introduction

3D graphics and WebGL integration enable immersive visual experiences in ArkUI applications. This guide explores 3D rendering, shader programming, and WebGL integration techniques for advanced graphics applications.

## WebGL Integration

### WebGL Context Management

```typescript
interface WebGLConfig {
  width: number;
  height: number;
  antialias: boolean;
  alpha: boolean;
  depth: boolean;
  stencil: boolean;
}

class WebGLManager {
  private gl: WebGLRenderingContext | null = null;
  private canvas: HTMLCanvasElement | null = null;
  private programs = new Map<string, WebGLProgram>();
  private buffers = new Map<string, WebGLBuffer>();
  private textures = new Map<string, WebGLTexture>();

  initialize(canvas: HTMLCanvasElement, config: WebGLConfig): boolean {
    this.canvas = canvas;
    this.canvas.width = config.width;
    this.canvas.height = config.height;

    try {
      this.gl = canvas.getContext("webgl", {
        antialias: config.antialias,
        alpha: config.alpha,
        depth: config.depth,
        stencil: config.stencil,
      });

      if (!this.gl) {
        console.error("WebGL not supported");
        return false;
      }

      this.setupDefaultState();
      return true;
    } catch (error) {
      console.error("WebGL initialization failed:", error);
      return false;
    }
  }

  createShaderProgram(
    vertexShaderSource: string,
    fragmentShaderSource: string,
    name: string
  ): WebGLProgram | null {
    if (!this.gl) return null;

    const vertexShader = this.compileShader(
      vertexShaderSource,
      this.gl.VERTEX_SHADER
    );
    const fragmentShader = this.compileShader(
      fragmentShaderSource,
      this.gl.FRAGMENT_SHADER
    );

    if (!vertexShader || !fragmentShader) return null;

    const program = this.gl.createProgram();
    if (!program) return null;

    this.gl.attachShader(program, vertexShader);
    this.gl.attachShader(program, fragmentShader);
    this.gl.linkProgram(program);

    if (!this.gl.getProgramParameter(program, this.gl.LINK_STATUS)) {
      console.error(
        "Program linking failed:",
        this.gl.getProgramInfoLog(program)
      );
      return null;
    }

    this.programs.set(name, program);
    return program;
  }

  private compileShader(source: string, type: number): WebGLShader | null {
    if (!this.gl) return null;

    const shader = this.gl.createShader(type);
    if (!shader) return null;

    this.gl.shaderSource(shader, source);
    this.gl.compileShader(shader);

    if (!this.gl.getShaderParameter(shader, this.gl.COMPILE_STATUS)) {
      console.error(
        "Shader compilation failed:",
        this.gl.getShaderInfoLog(shader)
      );
      this.gl.deleteShader(shader);
      return null;
    }

    return shader;
  }

  private setupDefaultState(): void {
    if (!this.gl) return;

    this.gl.enable(this.gl.DEPTH_TEST);
    this.gl.enable(this.gl.CULL_FACE);
    this.gl.clearColor(0.0, 0.0, 0.0, 1.0);
  }

  getProgram(name: string): WebGLProgram | null {
    return this.programs.get(name) || null;
  }

  createBuffer(
    name: string,
    data: Float32Array,
    usage = WebGLRenderingContext.STATIC_DRAW
  ): WebGLBuffer | null {
    if (!this.gl) return null;

    const buffer = this.gl.createBuffer();
    if (!buffer) return null;

    this.gl.bindBuffer(this.gl.ARRAY_BUFFER, buffer);
    this.gl.bufferData(this.gl.ARRAY_BUFFER, data, usage);

    this.buffers.set(name, buffer);
    return buffer;
  }

  clear(): void {
    if (!this.gl) return;
    this.gl.clear(this.gl.COLOR_BUFFER_BIT | this.gl.DEPTH_BUFFER_BIT);
  }

  getContext(): WebGLRenderingContext | null {
    return this.gl;
  }
}
```

### Shader System

```typescript
class ShaderLibrary {
  static vertexShaders = {
    basic: `
      attribute vec3 a_position;
      attribute vec3 a_normal;
      attribute vec2 a_texCoord;
      
      uniform mat4 u_modelViewMatrix;
      uniform mat4 u_projectionMatrix;
      uniform mat3 u_normalMatrix;
      
      varying vec3 v_normal;
      varying vec2 v_texCoord;
      varying vec3 v_position;
      
      void main() {
        vec4 position = u_modelViewMatrix * vec4(a_position, 1.0);
        v_position = position.xyz;
        v_normal = u_normalMatrix * a_normal;
        v_texCoord = a_texCoord;
        
        gl_Position = u_projectionMatrix * position;
      }
    `,

    skinned: `
      attribute vec3 a_position;
      attribute vec3 a_normal;
      attribute vec2 a_texCoord;
      attribute vec4 a_boneWeights;
      attribute vec4 a_boneIndices;
      
      uniform mat4 u_modelViewMatrix;
      uniform mat4 u_projectionMatrix;
      uniform mat3 u_normalMatrix;
      uniform mat4 u_boneMatrices[64];
      
      varying vec3 v_normal;
      varying vec2 v_texCoord;
      varying vec3 v_position;
      
      void main() {
        mat4 boneMatrix = a_boneWeights.x * u_boneMatrices[int(a_boneIndices.x)] +
                         a_boneWeights.y * u_boneMatrices[int(a_boneIndices.y)] +
                         a_boneWeights.z * u_boneMatrices[int(a_boneIndices.z)] +
                         a_boneWeights.w * u_boneMatrices[int(a_boneIndices.w)];
        
        vec4 skinnedPosition = boneMatrix * vec4(a_position, 1.0);
        vec4 position = u_modelViewMatrix * skinnedPosition;
        
        v_position = position.xyz;
        v_normal = u_normalMatrix * (boneMatrix * vec4(a_normal, 0.0)).xyz;
        v_texCoord = a_texCoord;
        
        gl_Position = u_projectionMatrix * position;
      }
    `,
  };

  static fragmentShaders = {
    phong: `
      precision mediump float;
      
      varying vec3 v_normal;
      varying vec2 v_texCoord;
      varying vec3 v_position;
      
      uniform vec3 u_lightPosition;
      uniform vec3 u_lightColor;
      uniform vec3 u_ambientColor;
      uniform vec3 u_diffuseColor;
      uniform vec3 u_specularColor;
      uniform float u_shininess;
      uniform sampler2D u_texture;
      
      void main() {
        vec3 normal = normalize(v_normal);
        vec3 lightDir = normalize(u_lightPosition - v_position);
        vec3 viewDir = normalize(-v_position);
        vec3 reflectDir = reflect(-lightDir, normal);
        
        // Ambient
        vec3 ambient = u_ambientColor;
        
        // Diffuse
        float diff = max(dot(normal, lightDir), 0.0);
        vec3 diffuse = diff * u_lightColor * u_diffuseColor;
        
        // Specular
        float spec = pow(max(dot(viewDir, reflectDir), 0.0), u_shininess);
        vec3 specular = spec * u_lightColor * u_specularColor;
        
        vec3 textureColor = texture2D(u_texture, v_texCoord).rgb;
        vec3 finalColor = (ambient + diffuse + specular) * textureColor;
        
        gl_FragColor = vec4(finalColor, 1.0);
      }
    `,

    pbr: `
      precision mediump float;
      
      varying vec3 v_normal;
      varying vec2 v_texCoord;
      varying vec3 v_position;
      
      uniform vec3 u_lightPositions[4];
      uniform vec3 u_lightColors[4];
      uniform vec3 u_viewPosition;
      uniform float u_metallic;
      uniform float u_roughness;
      uniform vec3 u_albedo;
      
      const float PI = 3.14159265359;
      
      float DistributionGGX(vec3 N, vec3 H, float roughness) {
        float a = roughness * roughness;
        float a2 = a * a;
        float NdotH = max(dot(N, H), 0.0);
        float NdotH2 = NdotH * NdotH;
        
        float num = a2;
        float denom = (NdotH2 * (a2 - 1.0) + 1.0);
        denom = PI * denom * denom;
        
        return num / denom;
      }
      
      float GeometrySchlickGGX(float NdotV, float roughness) {
        float r = (roughness + 1.0);
        float k = (r * r) / 8.0;
        
        float num = NdotV;
        float denom = NdotV * (1.0 - k) + k;
        
        return num / denom;
      }
      
      float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) {
        float NdotV = max(dot(N, V), 0.0);
        float NdotL = max(dot(N, L), 0.0);
        float ggx2 = GeometrySchlickGGX(NdotV, roughness);
        float ggx1 = GeometrySchlickGGX(NdotL, roughness);
        
        return ggx1 * ggx2;
      }
      
      vec3 fresnelSchlick(float cosTheta, vec3 F0) {
        return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
      }
      
      void main() {
        vec3 N = normalize(v_normal);
        vec3 V = normalize(u_viewPosition - v_position);
        
        vec3 F0 = vec3(0.04);
        F0 = mix(F0, u_albedo, u_metallic);
        
        vec3 Lo = vec3(0.0);
        for(int i = 0; i < 4; ++i) {
          vec3 L = normalize(u_lightPositions[i] - v_position);
          vec3 H = normalize(V + L);
          float distance = length(u_lightPositions[i] - v_position);
          float attenuation = 1.0 / (distance * distance);
          vec3 radiance = u_lightColors[i] * attenuation;
          
          float NDF = DistributionGGX(N, H, u_roughness);
          float G = GeometrySmith(N, V, L, u_roughness);
          vec3 F = fresnelSchlick(max(dot(H, V), 0.0), F0);
          
          vec3 kS = F;
          vec3 kD = vec3(1.0) - kS;
          kD *= 1.0 - u_metallic;
          
          vec3 numerator = NDF * G * F;
          float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0);
          vec3 specular = numerator / max(denominator, 0.001);
          
          float NdotL = max(dot(N, L), 0.0);
          Lo += (kD * u_albedo / PI + specular) * radiance * NdotL;
        }
        
        vec3 ambient = vec3(0.03) * u_albedo;
        vec3 color = ambient + Lo;
        
        // HDR tonemapping
        color = color / (color + vec3(1.0));
        // Gamma correction
        color = pow(color, vec3(1.0/2.2));
        
        gl_FragColor = vec4(color, 1.0);
      }
    `,
  };
}
```

### 3D Scene Management

```typescript
interface Transform {
  position: vec3;
  rotation: vec3;
  scale: vec3;
}

interface Material {
  diffuseColor: vec3;
  specularColor: vec3;
  shininess: number;
  metallic?: number;
  roughness?: number;
  albedo?: vec3;
  texture?: WebGLTexture;
}

interface Light {
  type: "directional" | "point" | "spot";
  position: vec3;
  direction?: vec3;
  color: vec3;
  intensity: number;
  range?: number;
  angle?: number;
}

class SceneObject {
  transform: Transform = {
    position: [0, 0, 0],
    rotation: [0, 0, 0],
    scale: [1, 1, 1],
  };

  material: Material = {
    diffuseColor: [1, 1, 1],
    specularColor: [1, 1, 1],
    shininess: 32,
  };

  geometry: Float32Array = new Float32Array();
  indices: Uint16Array = new Uint16Array();
  visible = true;

  getModelMatrix(): mat4 {
    const modelMatrix = mat4.create();
    mat4.translate(modelMatrix, modelMatrix, this.transform.position);
    mat4.rotateX(modelMatrix, modelMatrix, this.transform.rotation[0]);
    mat4.rotateY(modelMatrix, modelMatrix, this.transform.rotation[1]);
    mat4.rotateZ(modelMatrix, modelMatrix, this.transform.rotation[2]);
    mat4.scale(modelMatrix, modelMatrix, this.transform.scale);
    return modelMatrix;
  }
}

class Camera {
  position: vec3 = [0, 0, 5];
  target: vec3 = [0, 0, 0];
  up: vec3 = [0, 1, 0];
  fov = 45;
  aspect = 1;
  near = 0.1;
  far = 100;

  getViewMatrix(): mat4 {
    const viewMatrix = mat4.create();
    mat4.lookAt(viewMatrix, this.position, this.target, this.up);
    return viewMatrix;
  }

  getProjectionMatrix(): mat4 {
    const projectionMatrix = mat4.create();
    mat4.perspective(
      projectionMatrix,
      (this.fov * Math.PI) / 180,
      this.aspect,
      this.near,
      this.far
    );
    return projectionMatrix;
  }
}

class Scene {
  objects: SceneObject[] = [];
  lights: Light[] = [];
  camera = new Camera();
  backgroundColor: vec3 = [0, 0, 0];

  addObject(object: SceneObject): void {
    this.objects.push(object);
  }

  removeObject(object: SceneObject): void {
    const index = this.objects.indexOf(object);
    if (index >= 0) {
      this.objects.splice(index, 1);
    }
  }

  addLight(light: Light): void {
    this.lights.push(light);
  }

  getVisibleObjects(): SceneObject[] {
    return this.objects.filter((obj) => obj.visible);
  }
}
```

### 3D Renderer

```typescript
class Renderer3D {
  private webglManager: WebGLManager;
  private scene: Scene | null = null;
  private uniformLocations = new Map<
    string,
    Map<string, WebGLUniformLocation>
  >();

  constructor(webglManager: WebGLManager) {
    this.webglManager = webglManager;
  }

  setScene(scene: Scene): void {
    this.scene = scene;
  }

  render(): void {
    if (!this.scene) return;

    const gl = this.webglManager.getContext();
    if (!gl) return;

    this.webglManager.clear();

    const viewMatrix = this.scene.camera.getViewMatrix();
    const projectionMatrix = this.scene.camera.getProjectionMatrix();

    this.scene.getVisibleObjects().forEach((object) => {
      this.renderObject(object, viewMatrix, projectionMatrix);
    });
  }

  private renderObject(
    object: SceneObject,
    viewMatrix: mat4,
    projectionMatrix: mat4
  ): void {
    const gl = this.webglManager.getContext();
    if (!gl) return;

    const program = this.webglManager.getProgram("basic");
    if (!program) return;

    gl.useProgram(program);

    // Set uniforms
    const modelMatrix = object.getModelMatrix();
    const modelViewMatrix = mat4.create();
    mat4.multiply(modelViewMatrix, viewMatrix, modelMatrix);

    const normalMatrix = mat3.create();
    mat3.normalFromMat4(normalMatrix, modelViewMatrix);

    this.setUniform(program, "u_modelViewMatrix", modelViewMatrix);
    this.setUniform(program, "u_projectionMatrix", projectionMatrix);
    this.setUniform(program, "u_normalMatrix", normalMatrix);

    // Set material uniforms
    this.setUniform(program, "u_diffuseColor", object.material.diffuseColor);
    this.setUniform(program, "u_specularColor", object.material.specularColor);
    this.setUniform(program, "u_shininess", object.material.shininess);

    // Set lighting uniforms
    if (this.scene!.lights.length > 0) {
      const light = this.scene!.lights[0];
      this.setUniform(program, "u_lightPosition", light.position);
      this.setUniform(program, "u_lightColor", light.color);
    }

    // Bind geometry
    this.bindGeometry(program, object);

    // Draw
    if (object.indices.length > 0) {
      gl.drawElements(
        gl.TRIANGLES,
        object.indices.length,
        gl.UNSIGNED_SHORT,
        0
      );
    } else {
      gl.drawArrays(gl.TRIANGLES, 0, object.geometry.length / 8); // Assuming 8 floats per vertex
    }
  }

  private setUniform(program: WebGLProgram, name: string, value: any): void {
    const gl = this.webglManager.getContext();
    if (!gl) return;

    let programUniforms = this.uniformLocations.get(program.toString());
    if (!programUniforms) {
      programUniforms = new Map();
      this.uniformLocations.set(program.toString(), programUniforms);
    }

    let location = programUniforms.get(name);
    if (!location) {
      location = gl.getUniformLocation(program, name);
      if (location) {
        programUniforms.set(name, location);
      }
    }

    if (!location) return;

    if (typeof value === "number") {
      gl.uniform1f(location, value);
    } else if (Array.isArray(value)) {
      switch (value.length) {
        case 3:
          gl.uniform3fv(location, value);
          break;
        case 4:
          gl.uniform4fv(location, value);
          break;
        case 9:
          gl.uniformMatrix3fv(location, false, value);
          break;
        case 16:
          gl.uniformMatrix4fv(location, false, value);
          break;
      }
    }
  }

  private bindGeometry(program: WebGLProgram, object: SceneObject): void {
    const gl = this.webglManager.getContext();
    if (!gl) return;

    // Create or get buffer for this object's geometry
    const buffer = this.webglManager.createBuffer(
      `object_${object.constructor.name}`,
      object.geometry
    );
    if (!buffer) return;

    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);

    // Set up vertex attributes
    const positionLocation = gl.getAttribLocation(program, "a_position");
    const normalLocation = gl.getAttribLocation(program, "a_normal");
    const texCoordLocation = gl.getAttribLocation(program, "a_texCoord");

    if (positionLocation >= 0) {
      gl.enableVertexAttribArray(positionLocation);
      gl.vertexAttribPointer(positionLocation, 3, gl.FLOAT, false, 32, 0);
    }

    if (normalLocation >= 0) {
      gl.enableVertexAttribArray(normalLocation);
      gl.vertexAttribPointer(normalLocation, 3, gl.FLOAT, false, 32, 12);
    }

    if (texCoordLocation >= 0) {
      gl.enableVertexAttribArray(texCoordLocation);
      gl.vertexAttribPointer(texCoordLocation, 2, gl.FLOAT, false, 32, 24);
    }
  }
}

// Vector and matrix utility types (simplified)
type vec3 = [number, number, number];
type mat3 = Float32Array;
type mat4 = Float32Array;

// Simplified matrix math (would use actual math library in practice)
const mat4 = {
  create: (): mat4 => new Float32Array(16),
  translate: (out: mat4, a: mat4, v: vec3): mat4 => out,
  rotateX: (out: mat4, a: mat4, rad: number): mat4 => out,
  rotateY: (out: mat4, a: mat4, rad: number): mat4 => out,
  rotateZ: (out: mat4, a: mat4, rad: number): mat4 => out,
  scale: (out: mat4, a: mat4, v: vec3): mat4 => out,
  lookAt: (out: mat4, eye: vec3, center: vec3, up: vec3): mat4 => out,
  perspective: (
    out: mat4,
    fovy: number,
    aspect: number,
    near: number,
    far: number
  ): mat4 => out,
  multiply: (out: mat4, a: mat4, b: mat4): mat4 => out,
};

const mat3 = {
  create: (): mat3 => new Float32Array(9),
  normalFromMat4: (out: mat3, a: mat4): mat3 => out,
};
```

## ArkUI Integration Component

```typescript
@Component
export struct WebGL3DView {
  @State private canvasId: string = `webgl_${Date.now()}`
  private webglManager = new WebGLManager()
  private renderer = new Renderer3D(this.webglManager)
  private scene = new Scene()
  private animationId?: number

  build() {
    Column() {
      Web({ src: this.getCanvasHTML(), controller: new WebController() })
        .width('100%')
        .height('100%')
        .onPageEnd(() => {
          this.initializeWebGL()
        })
    }
  }

  aboutToDisappear() {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId)
    }
  }

  private getCanvasHTML(): string {
    return `data:text/html,
      <html>
        <body style="margin:0;padding:0;">
          <canvas id="${this.canvasId}" width="800" height="600" style="width:100%;height:100%;"></canvas>
        </body>
      </html>
    `
  }

  private initializeWebGL(): void {
    // This would need to be adapted for actual ArkUI Web component integration
    setTimeout(() => {
      const canvas = document.getElementById(this.canvasId) as HTMLCanvasElement
      if (canvas) {
        const success = this.webglManager.initialize(canvas, {
          width: 800,
          height: 600,
          antialias: true,
          alpha: false,
          depth: true,
          stencil: false
        })

        if (success) {
          this.setupShaders()
          this.setupScene()
          this.startRenderLoop()
        }
      }
    }, 100)
  }

  private setupShaders(): void {
    this.webglManager.createShaderProgram(
      ShaderLibrary.vertexShaders.basic,
      ShaderLibrary.fragmentShaders.phong,
      'basic'
    )
  }

  private setupScene(): void {
    // Create a simple cube
    const cube = new SceneObject()
    cube.geometry = this.createCubeGeometry()
    cube.material = {
      diffuseColor: [0.8, 0.2, 0.2],
      specularColor: [1.0, 1.0, 1.0],
      shininess: 32
    }
    this.scene.addObject(cube)

    // Add lighting
    this.scene.addLight({
      type: 'point',
      position: [2, 2, 2],
      color: [1, 1, 1],
      intensity: 1.0
    })

    // Set up camera
    this.scene.camera.position = [0, 0, 5]
    this.scene.camera.target = [0, 0, 0]

    this.renderer.setScene(this.scene)
  }

  private createCubeGeometry(): Float32Array {
    // Simplified cube vertices (position, normal, texCoord)
    return new Float32Array([
      // Front face
      -1, -1,  1,  0,  0,  1,  0, 0,
       1, -1,  1,  0,  0,  1,  1, 0,
       1,  1,  1,  0,  0,  1,  1, 1,
      -1,  1,  1,  0,  0,  1,  0, 1,
      // Add other faces...
    ])
  }

  private startRenderLoop(): void {
    const render = () => {
      // Rotate the cube
      if (this.scene.objects.length > 0) {
        this.scene.objects[0].transform.rotation[1] += 0.01
      }

      this.renderer.render()
      this.animationId = requestAnimationFrame(render)
    }
    render()
  }
}
```

## Conclusion

3D Graphics and WebGL integration in ArkTS enables:

- Advanced 3D rendering capabilities with WebGL
- Sophisticated shader programming for visual effects
- 3D scene management and object hierarchies
- Real-time lighting and material systems
- Integration with ArkUI component architecture
- Performance-optimized rendering pipelines

These techniques provide the foundation for creating immersive 3D experiences and advanced visual applications within HarmonyOS applications.
