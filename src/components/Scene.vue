<template>
  <div ref="container" class="webgl-container"></div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { defineComponent, onMounted, onUnmounted, watchEffect } from 'vue'

export default defineComponent({
  name: 'ThreeSketch',
  props: {
    clickCount: {
      type: Number,
      required: true,
      default: 0,
    },
  },
  setup(props) {
    let sketch = null

    class Sketch {
      constructor(container) {
        this.scene = new THREE.Scene()
        this.container = container
        this.width = container.offsetWidth
        this.height = container.offsetHeight
        this.temporaryPlanes = []

        this.renderer = new THREE.WebGLRenderer({
          antialias: true,
          alpha: true,
        })
        this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
        this.renderer.setSize(this.width, this.height)
        this.renderer.physicallyCorrectLights = true
        this.renderer.outputEncoding = THREE.sRGBEncoding

        container.appendChild(this.renderer.domElement)

        this.camera = new THREE.PerspectiveCamera(
          70,
          this.width / this.height,
          0.01,
          1000
        )
        this.camera.position.set(0, 0, 2)

        this.controls = new OrbitControls(this.camera, this.renderer.domElement)
        this.time = 0

        this.addObjects()
        this.setupClickHandler()
        this.resize()
        this.render()
      }

      getMaterial(opacity = 1.0) {
        return new THREE.ShaderMaterial({
          vertexShader: `
            varying vec2 vUv;
            void main() {
              vUv = uv;
              gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
          `,
          fragmentShader: `
            uniform float uTime;
            uniform float uClickCount;
            uniform float uOpacity;
            varying vec2 vUv;
            void main() {
              vec4 color = vec4(0.5, vUv, uOpacity);
              gl_FragColor = color;
            }
          `,
          side: THREE.DoubleSide,
          transparent: true,
          uniforms: {
            uTime: { type: 'f', value: 0 },
            uClickCount: { type: 'f', value: 0 },
            uOpacity: { type: 'f', value: opacity },
          },
        })
      }

      addObjects() {
        this.geometry = new THREE.PlaneGeometry(1, 1, 1, 1)
        this.material = this.getMaterial()
        this.mesh = new THREE.Mesh(this.geometry, this.material)
        this.scene.add(this.mesh)
      }

      setupClickHandler() {
        const raycaster = new THREE.Raycaster()
        const mouse = new THREE.Vector2()

        const handleClick = (event) => {
          // Prevent handling if OrbitControls is active
          if (this.controls.enabled && this.controls.isDragging) return

          // Calculate mouse position relative to the viewport
          const canvasRect = this.renderer.domElement.getBoundingClientRect()

          // Convert click coordinates to normalized device coordinates (-1 to +1)
          mouse.x =
            ((event.clientX - canvasRect.left) / canvasRect.width) * 2 - 1
          mouse.y =
            -((event.clientY - canvasRect.top) / canvasRect.height) * 2 + 1

          // Update the picking ray with the camera and mouse position
          raycaster.setFromCamera(mouse, this.camera)

          // Calculate the point of intersection with the z=0 plane
          const plane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 0)
          const intersectionPoint = new THREE.Vector3()
          raycaster.ray.intersectPlane(plane, intersectionPoint)

          // Create temporary plane at intersection point
          this.createTemporaryPlane(intersectionPoint)
        }

        // Store the handler for cleanup
        this.clickHandler = handleClick
        window.addEventListener('click', this.clickHandler)
      }

      createTemporaryPlane(position) {
        const smallGeometry = new THREE.PlaneGeometry(0.1, 0.1)
        const material = this.getMaterial(1.0)
        const plane = new THREE.Mesh(smallGeometry, material)

        plane.position.copy(position)
        plane.creationTime = this.time
        this.scene.add(plane)
        this.temporaryPlanes.push(plane)

        // Schedule cleanup after animation
        setTimeout(() => {
          const index = this.temporaryPlanes.indexOf(plane)
          if (index > -1) {
            this.temporaryPlanes.splice(index, 1)
            this.scene.remove(plane)
            plane.geometry.dispose()
            plane.material.dispose()
          }
        }, 500)
      }

      handleResize() {
        this.width = this.container.offsetWidth
        this.height = this.container.offsetHeight
        this.renderer.setSize(this.width, this.height)
        this.camera.aspect = this.width / this.height
        this.camera.updateProjectionMatrix()
      }

      resize() {
        window.addEventListener('resize', this.handleResize.bind(this))
      }

      render() {
        this.time += 0.01
        this.controls.update()

        // Update main plane
        this.material.uniforms.uTime.value = this.time
        this.material.uniforms.uClickCount.value = props.clickCount

        // Update temporary planes
        this.temporaryPlanes.forEach((plane) => {
          const age = this.time - plane.creationTime
          if (age < 0.2) {
            plane.material.uniforms.uTime.value = this.time
            plane.material.uniforms.uOpacity.value = 1.0 - age / 0.2
          }
        })

        requestAnimationFrame(this.render.bind(this))
        this.renderer.render(this.scene, this.camera)
      }

      cleanup() {
        // Remove event listeners
        window.removeEventListener('click', this.clickHandler)
        window.removeEventListener('resize', this.handleResize.bind(this))

        // Dispose of all temporary planes
        this.temporaryPlanes.forEach((plane) => {
          this.scene.remove(plane)
          plane.geometry.dispose()
          plane.material.dispose()
        })

        // Clear arrays
        this.temporaryPlanes = []

        // Dispose of main objects
        if (this.mesh) {
          this.scene.remove(this.mesh)
          this.geometry.dispose()
          this.material.dispose()
        }

        // Dispose of renderer and controls
        this.renderer.dispose()
        this.controls.dispose()
      }
    }

    onMounted(() => {
      const container = document.querySelector('.webgl-container')
      sketch = new Sketch(container)
    })

    onUnmounted(() => {
      if (sketch) {
        sketch.cleanup()
      }
    })

    return {}
  },
})
</script>

<style scoped lang="scss">
@import '../scss/variables.scss';
.webgl-container {
  position: absolute;
  z-index: -1;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100%;
  background-color: $black;
  canvas {
    display: block;
  }
}
</style>
