# 13 3D TEXT
* going to be using the `TextBufferGeometry` class but we need a particular font format called `typeface`
* if using a typeface that you've downloaded make sure you have the rights to use 
## HOW TO GET A TYPEFACE FONT
* can convert fonts with tools like `https://gero3.github.io/facetype.js/`
* can use fonts provided by Three.js 
* go to `/node_modules/three/examples/fonts/` folder, can take these and put in `/static/` folder or import them directly:
    ```
    import typefaceFont from 'three/examples/fonts/helvetiker_regular.typeface.json'
    ```
* mix the 2 methods:
    - open `/node_modules/three/examples/fonts/'
    - take `helvetiker_regular.typeface.jscon` and `LICENSE` files
    - put them in the `/static/fonts/` folder
* font now accessible at the `/fonts/helvetiker_regular.typeface.json` URL
## LOAD THE FONT
* need to use `FontLoader`:
    ```
    /**
    * Fonts
    */
    const fontLoader = new FontLoader()

    fontLoader.load(
        'fonts/helvetiker_regular.typeface.json',
        (font) => {
            console.log('loaded')
        }
    )
    ```
## CREATE GEOMETRY
* use `TextGeometry`:
    ```
    fontLoader.load(
        'fonts/helvetiker_regular.typeface.json',
        (font) => {
            const textGeometry = new TextGeometry(
                'Hello Three.js',
                {
                    font: font,
                    size: 0.5,
                    height: 0.2,
                    curveSegments: 12,
                    bevelEnabled: true, 
                    bevelThickness: 0.03,
                    bevelSize: 0.02,
                    bevelOffset: 0,
                    bevelSegments: 5
                }
            )
            const textMaterial = new THREE.MeshBasicMaterial()
            const text = new THREE.Mesh(textGeometry, textMaterial)
            scene.add(text)
        }
    )
    ```
* test geometry by adding `wireframe: true` to material:
    ```
    const textMaterial = new THREE.MeshBasicMaterial({ wireframe: true })
    ``` 
    or 
    ```
    textMaterial.wireframe = true
    ```
* creating a text geometry is long and hard for the computer
* avoid doing it too many times and keep the geometry as low poly as possible by reducing the `curveSegments` and `bevelSegments`
* remove the `wireframe` once happy with the level of details 
## CENTER THE TEXT
* add axis helper to help find center:
    ```
    // Axes helper
    const axesHelper = new THREE.AxesHelper()
    scene.add(axesHelper)
    ```
* multiple solutions to center text:
### USING THE BOUNDING
* the bounding is information associated with the geometry that tells what space is taken by that geometry
* it can be a box or sphere
* it helps Three.js calculate if the object is on the screen `(frustum culling)`
* we're going to use the bounding measures to recenter the geometry
* by default Three.js uses sphere bounding
* calculate the box bounding with `computeBoundingBox()`:
    ```
    textGeometry.computeBoundingBox()
    console.log(textGeometry.boundingBox)
    ```
* result is an instance of `Box3` with `min` and `max` properties
* the `min` isn't `0` b/c the `bevelThickness` and `bevelSize` 
* instead of moving the mesh, move the whole geometry with `translate(...)`:
    ```
    textGeometry.translate(
    - textGeometry.boundingBox.max.x * 0.5,
    - textGeometry.boundingBox.max.y * 0.5,
    - textGeometry.boundingBox.max.z * 0.5,

    )
    ```
* text looks centered but it's not b/c the `bevelThickness` and `bevelSize`:  
    ```
    textGeometry.translate(
        - (textGeometry.boundingBox.max.x - 0.02) * 0.5,
        - (textGeometry.boundingBox.max.y - 0.02) * 0.5,
        - (textGeometry.boundingBox.max.z - 0.03) * 0.5,

    )
    ```
* this was the hard way
### USING CENTER()
* call the `center()` method on geometry:
    ```
    textGeometry.center()
    ```
## ADD A MATCAP MATERIAL
* going to use `MeshMatcapMaterial`
* can use the matcaps in the `/static/textures/matcaps/` folder or download one `https://github.io/nidorx/matcaps`
* load the texture with `TextureLoader`:
    ```
    const textureLoader = new THREE.TextureLoader()
    const matcapTexture = textureLoader.load('/textures/matcaps/1.png')
    // console.log(matcapTexture)
    ```
* replace `MeshBasicMaterial` with `MeshMatapMaterial` and use `matcapTexture` variable with the `matcaps` property:
    ```
    const textMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })
    ```
    or
    ```
    ```
    textMaterial.matcap = matcapTexture
    ```
## ADD OBJECTS
* in function create 100 donuts:
    ```
    for(let i = 0; i < 100; i++){
        const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)
        const donutMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })
        const donut = new THREE.Mesh(donutGeometry, donutMaterial)
        scene.add(donut)
    }
    ```
* add randomness in their position:
    ```
    for(let i = 0; i < 100; i++){
        const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)
        const donutMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })
        const donut = new THREE.Mesh(donutGeometry, donutMaterial)

        donut.position.x = (Math.random() - 0.5) * 10
        donut.position.y = (Math.random() - 0.5) * 10
        donut.position.z = (Math.random() - 0.5) * 10

        scene.add(donut)
    }
    ```
* add randomness in their rotation:
    ```
    donut.rotation.x = Math.random() * Math.PI
    donut.rotation.y = Math.random() * Math.PI
    ```
* add randomness in their scale:
    ```
    const scale = Math.random()
    donut.scale.set(scale, scale, scale)
    ```
## OPTIMIZE
    ```
    // before loop
    console.time('donuts')

    // after loop ends
    console.timeEnd('donuts')
    ```
* shows time taken to create donuts 
* took quater of a second which is too long 
* can use the same material and the geometry on multiple `Meshes`
* move `donutGeometry` and `donutMaterial` outside the loop so only creating 1 not 100
    ```
    const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)
    const donutMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })
    ```
* use the same material for the `text` 
* remove the `donutMaterial`, rename the `textMaterial` by `material` and use it for both `text` and `donut`:   
