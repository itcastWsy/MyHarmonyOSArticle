# ArkUI Advanced Graphics Rendering

## Introduction

Advanced Graphics Rendering in ArkUI enables high-performance 2D/3D graphics, shader programming, and real-time visual effects. This guide covers WebGL integration, custom rendering pipelines, and GPU-accelerated computations.

## Graphics Rendering Framework

```typescript
interface RenderContext {
  canvas: HTMLCanvasElement;
  gl: WebGLRenderingContext | WebGL2RenderingContext;
  shaders: Map<string, ShaderProgram>;
  buffers: Map<string, WebGLBuffer>;
  textures: Map<string, WebGLTexture>;
  framebuffers: Map<string, WebGLFramebuffer>;
}

interface ShaderProgram {
  id: string;
  program: WebGLProgram;
  vertexShader: WebGLShader;
  fragmentShader: WebGLShader;
  uniforms: Map<string, WebGLUniformLocation>;
  attributes: Map<string, number>;
}

class GraphicsRenderer {
  private context: RenderContext;
  private renderQueue: RenderCommand[] = [];
  private materials: Map<string, Material> = new Map();
  private meshes: Map<string, Mesh> = new Map();

  constructor(canvas: HTMLCanvasElement) {
    this.context = this.initializeContext(canvas);
    this.setupDefaultShaders();
  }

  createShader(
    id: string,
    vertexSource: string,
    fragmentSource: string
  ): boolean {
    const gl = this.context.gl;

    try {
      const vertexShader = this.compileShader(gl.VERTEX_SHADER, vertexSource);
      const fragmentShader = this.compileShader(
        gl.FRAGMENT_SHADER,
        fragmentSource
      );

      const program = gl.createProgram()!;
      gl.attachShader(program, vertexShader);
      gl.attachShader(program, fragmentShader);
      gl.linkProgram(program);

      if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
        console.error(
          "Shader program linking failed:",
          gl.getProgramInfoLog(program)
        );
        return false;
      }

      const shaderProgram: ShaderProgram = {
        id,
        program,
        vertexShader,
        fragmentShader,
        uniforms: new Map(),
        attributes: new Map(),
      };

      this.extractUniformsAndAttributes(shaderProgram);
      this.context.shaders.set(id, shaderProgram);

      return true;
    } catch (error) {
      console.error("Shader creation failed:", error);
      return false;
    }
  }

  createMesh(
    id: string,
    vertices: number[],
    indices: number[],
    normals?: number[],
    uvs?: number[]
  ): void {
    const gl = this.context.gl;

    const vertexBuffer = gl.createBuffer()!;
    gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);

    const indexBuffer = gl.createBuffer()!;
    gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indexBuffer);
    gl.bufferData(
      gl.ELEMENT_ARRAY_BUFFER,
      new Uint16Array(indices),
      gl.STATIC_DRAW
    );

    const mesh: Mesh = {
      id,
      vertexBuffer,
      indexBuffer,
      vertexCount: vertices.length / 3,
      indexCount: indices.length,
      normalBuffer: null,
      uvBuffer: null,
    };

    if (normals) {
      mesh.normalBuffer = gl.createBuffer()!;
      gl.bindBuffer(gl.ARRAY_BUFFER, mesh.normalBuffer);
      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(normals), gl.STATIC_DRAW);
    }

    if (uvs) {
      mesh.uvBuffer = gl.createBuffer()!;
      gl.bindBuffer(gl.ARRAY_BUFFER, mesh.uvBuffer);
      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(uvs), gl.STATIC_DRAW);
    }

    this.meshes.set(id, mesh);
    this.context.buffers.set(`${id}_vertices`, vertexBuffer);
    this.context.buffers.set(`${id}_indices`, indexBuffer);
  }

  createTexture(
    id: string,
    image: ImageData | HTMLImageElement
  ): WebGLTexture | null {
    const gl = this.context.gl;
    const texture = gl.createTexture();

    if (!texture) return null;

    gl.bindTexture(gl.TEXTURE_2D, texture);

    if (image instanceof ImageData) {
      gl.texImage2D(
        gl.TEXTURE_2D,
        0,
        gl.RGBA,
        gl.RGBA,
        gl.UNSIGNED_BYTE,
        image
      );
    } else {
      gl.texImage2D(
        gl.TEXTURE_2D,
        0,
        gl.RGBA,
        gl.RGBA,
        gl.UNSIGNED_BYTE,
        image
      );
    }

    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);

    this.context.textures.set(id, texture);
    return texture;
  }

  createMaterial(id: string, config: MaterialConfig): void {
    const material: Material = {
      id,
      shaderId: config.shaderId,
      uniforms: new Map(Object.entries(config.uniforms || {})),
      textures: new Map(Object.entries(config.textures || {})),
      blendMode: config.blendMode || "normal",
      cullFace: config.cullFace || "back",
      depthTest: config.depthTest !== false,
    };

    this.materials.set(id, material);
  }

  renderMesh(meshId: string, materialId: string, transform: Transform): void {
    const command: RenderCommand = {
      type: "mesh",
      meshId,
      materialId,
      transform,
      timestamp: performance.now(),
    };

    this.renderQueue.push(command);
  }

  renderPostEffect(effectId: string, input: string, output?: string): void {
    const command: RenderCommand = {
      type: "postEffect",
      effectId,
      input,
      output,
      timestamp: performance.now(),
    };

    this.renderQueue.push(command);
  }

  flush(): void {
    const gl = this.context.gl;

    // Sort render queue by material to minimize state changes
    this.renderQueue.sort((a, b) => {
      if (a.materialId < b.materialId) return -1;
      if (a.materialId > b.materialId) return 1;
      return 0;
    });

    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

    let currentMaterial: string | null = null;

    for (const command of this.renderQueue) {
      if (command.type === "mesh") {
        this.executeMeshCommand(command, currentMaterial);
        currentMaterial = command.materialId;
      } else if (command.type === "postEffect") {
        this.executePostEffectCommand(command);
      }
    }

    this.renderQueue = [];
  }

  private initializeContext(canvas: HTMLCanvasElement): RenderContext {
    const gl = canvas.getContext("webgl2") || canvas.getContext("webgl");

    if (!gl) {
      throw new Error("WebGL not supported");
    }

    gl.enable(gl.DEPTH_TEST);
    gl.enable(gl.CULL_FACE);
    gl.clearColor(0.0, 0.0, 0.0, 1.0);

    return {
      canvas,
      gl,
      shaders: new Map(),
      buffers: new Map(),
      textures: new Map(),
      framebuffers: new Map(),
    };
  }

  private compileShader(type: number, source: string): WebGLShader {
    const gl = this.context.gl;
    const shader = gl.createShader(type)!;

    gl.shaderSource(shader, source);
    gl.compileShader(shader);

    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
      const error = gl.getShaderInfoLog(shader);
      gl.deleteShader(shader);
      throw new Error(`Shader compilation failed: ${error}`);
    }

    return shader;
  }

  private extractUniformsAndAttributes(shader: ShaderProgram): void {
    const gl = this.context.gl;

    // Extract uniforms
    const uniformCount = gl.getProgramParameter(
      shader.program,
      gl.ACTIVE_UNIFORMS
    );
    for (let i = 0; i < uniformCount; i++) {
      const uniform = gl.getActiveUniform(shader.program, i)!;
      const location = gl.getUniformLocation(shader.program, uniform.name)!;
      shader.uniforms.set(uniform.name, location);
    }

    // Extract attributes
    const attributeCount = gl.getProgramParameter(
      shader.program,
      gl.ACTIVE_ATTRIBUTES
    );
    for (let i = 0; i < attributeCount; i++) {
      const attribute = gl.getActiveAttrib(shader.program, i)!;
      const location = gl.getAttribLocation(shader.program, attribute.name);
      shader.attributes.set(attribute.name, location);
    }
  }

  private executeMeshCommand(
    command: RenderCommand,
    currentMaterial: string | null
  ): void {
    const gl = this.context.gl;
    const mesh = this.meshes.get(command.meshId!);
    const material = this.materials.get(command.materialId!);

    if (!mesh || !material) return;

    // Bind shader if material changed
    if (currentMaterial !== command.materialId) {
      const shader = this.context.shaders.get(material.shaderId);
      if (!shader) return;

      gl.useProgram(shader.program);
      this.bindMaterialUniforms(material, shader);
    }

    // Bind vertex data
    this.bindMeshData(mesh);

    // Set transform uniforms
    this.setTransformUniforms(command.transform!);

    // Draw
    gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, mesh.indexBuffer);
    gl.drawElements(gl.TRIANGLES, mesh.indexCount, gl.UNSIGNED_SHORT, 0);
  }

  private bindMaterialUniforms(
    material: Material,
    shader: ShaderProgram
  ): void {
    const gl = this.context.gl;

    // Bind textures
    let textureUnit = 0;
    for (const [name, textureId] of material.textures) {
      const texture = this.context.textures.get(textureId);
      if (texture) {
        gl.activeTexture(gl.TEXTURE0 + textureUnit);
        gl.bindTexture(gl.TEXTURE_2D, texture);

        const location = shader.uniforms.get(name);
        if (location) {
          gl.uniform1i(location, textureUnit);
        }
        textureUnit++;
      }
    }

    // Bind uniform values
    for (const [name, value] of material.uniforms) {
      const location = shader.uniforms.get(name);
      if (location) {
        this.setUniformValue(location, value);
      }
    }
  }

  private setUniformValue(location: WebGLUniformLocation, value: any): void {
    const gl = this.context.gl;

    if (typeof value === "number") {
      gl.uniform1f(location, value);
    } else if (Array.isArray(value)) {
      switch (value.length) {
        case 2:
          gl.uniform2fv(location, value);
          break;
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

  private setupDefaultShaders(): void {
    const basicVertexShader = `
      attribute vec3 a_position;
      attribute vec3 a_normal;
      attribute vec2 a_uv;
      
      uniform mat4 u_modelMatrix;
      uniform mat4 u_viewMatrix;
      uniform mat4 u_projectionMatrix;
      
      varying vec3 v_normal;
      varying vec2 v_uv;
      
      void main() {
        gl_Position = u_projectionMatrix * u_viewMatrix * u_modelMatrix * vec4(a_position, 1.0);
        v_normal = (u_modelMatrix * vec4(a_normal, 0.0)).xyz;
        v_uv = a_uv;
      }
    `;

    const basicFragmentShader = `
      precision mediump float;
      
      varying vec3 v_normal;
      varying vec2 v_uv;
      
      uniform vec3 u_color;
      uniform sampler2D u_texture;
      
      void main() {
        vec3 color = u_color * texture2D(u_texture, v_uv).rgb;
        gl_FragColor = vec4(color, 1.0);
      }
    `;

    this.createShader("basic", basicVertexShader, basicFragmentShader);
  }
}

interface Mesh {
  id: string;
  vertexBuffer: WebGLBuffer;
  indexBuffer: WebGLBuffer;
  normalBuffer: WebGLBuffer | null;
  uvBuffer: WebGLBuffer | null;
  vertexCount: number;
  indexCount: number;
}

interface Material {
  id: string;
  shaderId: string;
  uniforms: Map<string, any>;
  textures: Map<string, string>;
  blendMode: BlendMode;
  cullFace: CullFace;
  depthTest: boolean;
}

interface RenderCommand {
  type: "mesh" | "postEffect";
  meshId?: string;
  materialId?: string;
  effectId?: string;
  transform?: Transform;
  input?: string;
  output?: string;
  timestamp: number;
}

interface Transform {
  position: [number, number, number];
  rotation: [number, number, number];
  scale: [number, number, number];
}

type BlendMode = "normal" | "additive" | "multiply" | "screen";
type CullFace = "none" | "front" | "back";
```

## High-Performance Graphics Component

```typescript
@Component
struct AdvancedGraphicsRenderer {
  @State private meshes: string[] = []
  @State private materials: string[] = []
  @State private renderStats: RenderStats = {
    frameTime: 0,
    drawCalls: 0,
    triangles: 0,
    fps: 0
  }

  private canvas: HTMLCanvasElement | null = null
  private renderer: GraphicsRenderer | null = null
  private animationId: number = 0

  aboutToAppear() {
    this.initializeRenderer()
  }

  aboutToDisappear() {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId)
    }
  }

  build() {
    Column() {
      this.buildHeader()
      this.buildCanvas()
      this.buildControls()
      this.buildStats()
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Advanced Graphics Renderer')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      Text(`FPS: ${this.renderStats.fps.toFixed(1)}`)
        .fontSize(14)
        .fontColor('#007AFF')
        .backgroundColor('#F0F8FF')
        .padding({ horizontal: 8, vertical: 4 })
        .borderRadius(4)
    }
    .margin({ bottom: 16 })
    .padding({ horizontal: 16, top: 16 })
  }

  @Builder
  private buildCanvas() {
    // Canvas placeholder - in real implementation would use custom component
    Row() {
      Text('WebGL Canvas')
        .fontSize(16)
        .fontColor('#666666')
    }
    .width('100%')
    .height(400)
    .backgroundColor('#000000')
    .borderRadius(8)
    .justifyContent(FlexAlign.Center)
    .margin({ horizontal: 16 })
  }

  @Builder
  private buildControls() {
    Column() {
      Text('Rendering Controls')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Grid() {
        GridItem() {
          Button('Add Cube')
            .onClick(() => this.addCube())
            .backgroundColor('#007AFF')
            .fontColor('#FFFFFF')
            .width('100%')
        }

        GridItem() {
          Button('Add Sphere')
            .onClick(() => this.addSphere())
            .backgroundColor('#34C759')
            .fontColor('#FFFFFF')
            .width('100%')
        }

        GridItem() {
          Button('Add Plane')
            .onClick(() => this.addPlane())
            .backgroundColor('#FF9500')
            .fontColor('#FFFFFF')
            .width('100%')
        }

        GridItem() {
          Button('Clear Scene')
            .onClick(() => this.clearScene())
            .backgroundColor('#FF3B30')
            .fontColor('#FFFFFF')
            .width('100%')
        }
      }
      .columnsTemplate('1fr 1fr')
      .columnsGap(12)
      .rowsGap(12)
      .margin({ bottom: 16 })

      Row() {
        Button('Start Animation')
          .onClick(() => this.startAnimation())
          .backgroundColor('#8E44AD')
          .fontColor('#FFFFFF')
          .flexGrow(1)
          .margin({ right: 8 })

        Button('Stop Animation')
          .onClick(() => this.stopAnimation())
          .backgroundColor('#8E8E93')
          .fontColor('#FFFFFF')
          .flexGrow(1)
      }
    }
    .alignItems(HorizontalAlign.Start)
    .padding({ horizontal: 16 })
    .margin({ top: 16 })
  }

  @Builder
  private buildStats() {
    Column() {
      Text('Render Statistics')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Grid() {
        GridItem() {
          this.buildStatCard('Frame Time', `${this.renderStats.frameTime.toFixed(2)}ms`)
        }

        GridItem() {
          this.buildStatCard('Draw Calls', this.renderStats.drawCalls.toString())
        }

        GridItem() {
          this.buildStatCard('Triangles', this.renderStats.triangles.toString())
        }

        GridItem() {
          this.buildStatCard('Meshes', this.meshes.length.toString())
        }
      }
      .columnsTemplate('1fr 1fr')
      .columnsGap(12)
      .rowsGap(12)
    }
    .alignItems(HorizontalAlign.Start)
    .padding(16)
    .margin({ top: 16 })
  }

  @Builder
  private buildStatCard(title: string, value: string) {
    Column() {
      Text(title)
        .fontSize(12)
        .fontColor('#666666')
        .margin({ bottom: 4 })

      Text(value)
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .fontColor('#007AFF')
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(6)
    .alignItems(HorizontalAlign.Center)
  }

  private initializeRenderer(): void {
    // In real implementation, get canvas from custom component
    // this.canvas = canvasElement
    // this.renderer = new GraphicsRenderer(this.canvas)

    // Create default materials
    this.createDefaultMaterials()

    // Start render loop
    this.startRenderLoop()
  }

  private createDefaultMaterials(): void {
    if (!this.renderer) return

    this.renderer.createMaterial('red', {
      shaderId: 'basic',
      uniforms: { u_color: [1.0, 0.0, 0.0] }
    })

    this.renderer.createMaterial('green', {
      shaderId: 'basic',
      uniforms: { u_color: [0.0, 1.0, 0.0] }
    })

    this.renderer.createMaterial('blue', {
      shaderId: 'basic',
      uniforms: { u_color: [0.0, 0.0, 1.0] }
    })

    this.materials = ['red', 'green', 'blue']
  }

  private addCube(): void {
    if (!this.renderer) return

    const cubeVertices = [
      // Front face
      -1, -1,  1,   1, -1,  1,   1,  1,  1,  -1,  1,  1,
      // Back face
      -1, -1, -1,  -1,  1, -1,   1,  1, -1,   1, -1, -1,
      // Top face
      -1,  1, -1,  -1,  1,  1,   1,  1,  1,   1,  1, -1,
      // Bottom face
      -1, -1, -1,   1, -1, -1,   1, -1,  1,  -1, -1,  1,
      // Right face
       1, -1, -1,   1,  1, -1,   1,  1,  1,   1, -1,  1,
      // Left face
      -1, -1, -1,  -1, -1,  1,  -1,  1,  1,  -1,  1, -1
    ]

    const cubeIndices = [
      0,  1,  2,    0,  2,  3,    // front
      4,  5,  6,    4,  6,  7,    // back
      8,  9, 10,    8, 10, 11,    // top
      12,13, 14,   12, 14, 15,    // bottom
      16,17, 18,   16, 18, 19,    // right
      20,21, 22,   20, 22, 23     // left
    ]

    const meshId = `cube_${Date.now()}`
    this.renderer.createMesh(meshId, cubeVertices, cubeIndices)
    this.meshes.push(meshId)
  }

  private addSphere(): void {
    if (!this.renderer) return

    const { vertices, indices } = this.generateSphereGeometry(1.0, 16, 16)
    const meshId = `sphere_${Date.now()}`

    this.renderer.createMesh(meshId, vertices, indices)
    this.meshes.push(meshId)
  }

  private addPlane(): void {
    if (!this.renderer) return

    const planeVertices = [
      -2, 0, -2,   2, 0, -2,   2, 0,  2,  -2, 0,  2
    ]

    const planeIndices = [0, 1, 2,   0, 2, 3]

    const meshId = `plane_${Date.now()}`
    this.renderer.createMesh(meshId, planeVertices, planeIndices)
    this.meshes.push(meshId)
  }

  private clearScene(): void {
    this.meshes = []
  }

  private startAnimation(): void {
    if (this.animationId) return

    this.animationId = requestAnimationFrame(() => this.animate())
  }

  private stopAnimation(): void {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId)
      this.animationId = 0
    }
  }

  private animate(): void {
    const time = performance.now()

    // Render scene
    this.renderScene(time)

    // Continue animation
    this.animationId = requestAnimationFrame(() => this.animate())
  }

  private renderScene(time: number): void {
    if (!this.renderer) return

    const startTime = performance.now()

    // Render each mesh with animation
    this.meshes.forEach((meshId, index) => {
      const materialId = this.materials[index % this.materials.length]

      const transform: Transform = {
        position: [
          Math.sin(time * 0.001 + index) * 3,
          Math.cos(time * 0.002 + index) * 2,
          -5 + index * 2
        ],
        rotation: [
          time * 0.001 + index,
          time * 0.002 + index,
          0
        ],
        scale: [1, 1, 1]
      }

      this.renderer.renderMesh(meshId, materialId, transform)
    })

    this.renderer.flush()

    // Update stats
    const frameTime = performance.now() - startTime
    this.updateRenderStats(frameTime)
  }

  private startRenderLoop(): void {
    const renderLoop = () => {
      this.renderScene(performance.now())
      requestAnimationFrame(renderLoop)
    }

    requestAnimationFrame(renderLoop)
  }

  private generateSphereGeometry(radius: number, widthSegments: number, heightSegments: number) {
    const vertices: number[] = []
    const indices: number[] = []

    for (let j = 0; j <= heightSegments; j++) {
      const theta = j * Math.PI / heightSegments
      const sinTheta = Math.sin(theta)
      const cosTheta = Math.cos(theta)

      for (let i = 0; i <= widthSegments; i++) {
        const phi = i * 2 * Math.PI / widthSegments
        const sinPhi = Math.sin(phi)
        const cosPhi = Math.cos(phi)

        const x = cosPhi * sinTheta
        const y = cosTheta
        const z = sinPhi * sinTheta

        vertices.push(radius * x, radius * y, radius * z)
      }
    }

    for (let j = 0; j < heightSegments; j++) {
      for (let i = 0; i < widthSegments; i++) {
        const first = (j * (widthSegments + 1)) + i
        const second = first + widthSegments + 1

        indices.push(first, second, first + 1)
        indices.push(second, second + 1, first + 1)
      }
    }

    return { vertices, indices }
  }

  private updateRenderStats(frameTime: number): void {
    this.renderStats = {
      frameTime,
      drawCalls: this.meshes.length,
      triangles: this.meshes.length * 12, // Approximation
      fps: 1000 / frameTime
    }
  }
}

interface RenderStats {
  frameTime: number
  drawCalls: number
  triangles: number
  fps: number
}

interface MaterialConfig {
  shaderId: string
  uniforms?: Record<string, any>
  textures?: Record<string, string>
  blendMode?: BlendMode
  cullFace?: CullFace
  depthTest?: boolean
}
```

## Conclusion

Advanced Graphics Rendering in ArkUI provides:

- High-performance WebGL integration
- Custom shader programming capabilities
- Efficient rendering pipeline management
- Real-time 3D graphics and animations
- GPU-accelerated visual effects

These capabilities enable immersive visual experiences, data visualization, and high-performance gaming applications.
