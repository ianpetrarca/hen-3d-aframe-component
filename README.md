| | | |
|:-------------------------:|:-------------------------:|:-------------------------:|
|<img width="1604" alt="screen shot 2017-08-07 at 12 18 15 pm" src="https://user-images.githubusercontent.com/1003196/122152405-3d2bd680-ce2f-11eb-9937-b3951e2451ad.gif">  |  <img width="1604" alt="screen shot 2017-08-07 at 12 18 15 pm" src="https://user-images.githubusercontent.com/1003196/122152411-3f8e3080-ce2f-11eb-9aa5-6ba24913c7bb.gif">|<img width="1604" alt="screen shot 2017-08-07 at 12 18 15 pm" src="https://user-images.githubusercontent.com/1003196/122152418-4321b780-ce2f-11eb-83e0-17a77191f08a.gif">|


# Hic et nunc 3D Model Aframe Component

### Use 3D Models from [hic et nunc](hicetnunc.xyz/) in your Aframe WebXR Scenes!
#### hic et nunc is a decentralized NFT marketplace built on the Tezos blockchain. This Github repo is a guide for using user-created 3D Models in WebXR experiences using [Aframe](https://aframe.io). This comeponent uses a REST request message described in this [API Guide](https://github.com/ianpetrarca/hicetnunc_api_guide).

####

This component is added to GLTF model Aframe entity 
```html
      <a-gltf-model hen-model="
                    scale:3;
                    animated:true;
                    reflection:true;
                    url:https://www.hicetnunc.xyz/objkt/128211">
      </a-gltf-model>

```

#### Code Examples

- [Glitch.me Example](https://glitch.com/edit/#!/hen-model-aframe-component) 

#### CDN Link

```html
 <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script> 
 <script src="https://cdn.jsdelivr.net/npm/regenerator-runtime@0.13.7/runtime.min.js"></script>
 <script src="https://cdn.jsdelivr.net/gh/ianpetrarca/hen-model-aframe-component@main/src/js/components/hen.js"></script>
```

#### Using the Component 

Axios and Regenerator Runtime are required to use the API portion of this component**

```html
<html>
  <head>
    <!--  API Scripts    -->
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script> 
    <script src="https://cdn.jsdelivr.net/npm/regenerator-runtime@0.13.7/runtime.min.js"></script>
    <!--  Aframe and Aframe Extras Library    -->
    <script src="https://aframe.io/releases/1.2.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/gh/donmccurdy/aframe-extras@v6.1.1/dist/aframe-extras.min.js"></script>
    <!--  Hen-Aframe Component Hosted on CDN    -->
     <script src="https://cdn.jsdelivr.net/gh/ianpetrarca/hen-model-aframe-component@main/src/js/components/hen.js"></script>
  </head>
  <body>
    <!--  Aframe Scene with color management and high refresh rate enabled  -->
    <a-scene renderer="antialias: true;colorManagement: true;physicallyCorrectLights: true;highRefreshRate:true">
      <!-- Camera, Lighting and Sky -->
      <a-entity light="type: ambient; color: #BBB;intensity:1"></a-entity>
      <a-entity light="type: directional; color: #FFF; intensity: 0.6" position="-0.5 1 1"></a-entity>
      <a-camera position="0 1.6 0" look-controls="pointerLockEnabled:true" fov="50"></a-camera>
      <a-sky color="black"></a-sky>
      <!-- Hen-3D Component       -->
      <a-gltf-model hen-model="scale:3;animated:true;reflection:true;url:https://www.hicetnunc.xyz/objkt/128211" 
      position="0 1 -3"> </a-gltf-model>
     </a-scene>
  </body>
</html>
```

#### Component Schema
| Name | Function | Type | Default |
| :---         |     :---:      |          ---: |      ---: | 
| URL   | hic et nunc URL or OBJKT ID     | String    |       |
| Reflection     | Adds Cubemap Reflection      | Boolean      | True      |
| Scale     | Scale of Model     | Number      | 1      |
| Animated     | Enable Model Animation    | Boolean      | True      |

```js

//ASYNC REST API REQUEST
async function getTokenInfo(id){
    try {
        if(id.includes('xyz')){
          id = id.substring(32)
        }
        const res = await axios.get('https://api.better-call.dev/v1/tokens/mainnet/metadata?token_id=' + id.toString())
        return res.data[0]
    } catch (error) {
        return null
    }
  }
  
  AFRAME.registerComponent('hen-model', {
    schema: {
      url: {type:'string'}, //Hicetnunc.xyz URL or OBJKT ID input
      scale: {type:'number', default: 1}, //Scale of Model
      animated: {type:'boolean', default:true}, //Animate model
      reflection: {type:'boolean', default:true} //Animate model
    },
    init: function () {
  
      //Async Function to get IPFS Hash for OBJKT
      getTokenInfo(this.data.url).then((response) => {  
        let id = response.artifact_uri
        this.el.setAttribute('src','https://cloudflare-ipfs.com/ipfs/'+ id.slice(7,id.length))
      }).catch((response) => { 
          
      })
  
      //Turn on / off animation based on user input
      if(!this.data.animated){
        this.el.removeAttribute('animation-mixer');
      } else {
        this.el.setAttribute('animation-mixer', '')
      }

      if(!this.data.reflection){

      }else {
        this.el.setAttribute('cube-env-map', 'path: https://storage.googleapis.com/titanpointe/indoor/; extension: png; reflectivity: 1')
      }


    
      // Variables for capturing the roughness/metalness values of each material in the model
      this.metalMap = {}
      this.roughMap = {}
  
      //Functions for Material parameters
      this.prepareMap.bind(this) //Parse Materials
      this.traverseMesh.bind(this) //Apply Materials 
      
      //Loading Text
      var text = document.createElement('a-text');
      text.setAttribute('value','Loading...')
      text.setAttribute('position', {x: -.5, y: 0, z: 0});
      this.el.appendChild(text)

      //Loading Spinner Box
      var loadBox = document.createElement('a-entity')
      loadBox.setAttribute('geometry', {
        primitive: 'box',
        height: .25,
        width: .25,
        depth:.25
      });

      //Add Loadbox Animations
      loadBox.setAttribute('position', {x: 0, y: .5, z: 0});
      loadBox.setAttribute('material','wireframe','true');
      loadBox.setAttribute('animation', 'property: rotation; to: 0 360 360;loop:true;dir:alternate;dur:3000');
      this.el.appendChild(loadBox)

      this.el.addEventListener('model-loaded', evt => 
       {
        this.el.removeChild(text)
        this.el.removeChild(loadBox)
        this.scale();
        this.prepareMap()
        this.update()
      });

    },
    prepareMap: function() { //Parse Model Nodes for material information
      this.traverseMesh(node => {
          this.roughMap[node.uuid] = node.material.roughness
          this.metalMap[node.uuid] = node.material.metalness
      })
    },
    update: function () { //Traverse Mesh and apply material parameters and cubemap
      this.traverseMesh(node => {
        node.material.metalness = this.metalMap[node.uuid]
        node.material.roughness = this.roughMap[node.uuid] 
        
        if(!node.material.envMapIntensity==1){ //Check for unlit models that dont require cubemap
         this.el.removeAttribute('cube-env-map');
        }
  
      })
    },
    traverseMesh: function(func) { 
      //Traverse nodes and find actual meshes in GLTF/GLB File
      var mesh = this.el.getObject3D('mesh');
      if (!mesh) { return; }
       mesh.traverse(node => {
        if (node.isMesh) {
          func(node)
        }
      }); 
    }, 
    scale: function () {
      //Get Object Meshes
      const el = this.el;
      const span = this.data.scale;
      const mesh = el.getObject3D('mesh');
  
      if (!mesh) return;
  
      // Compute bounds.
      const bbox = new THREE.Box3().setFromObject(mesh);
  
      // Normalize scale.
      const scale = span / bbox.getSize().length();
      mesh.scale.set(scale, scale, scale);
  
      //Normalize Position
      mesh.position.set(0,0,0);
    }
  });
    
  

```

#### Hic et nunc Resources
- [Hic et nunc Tools](hicetnunc.tools/)
- [Hic et nunc Discord](https://discord.gg/g7VQt5pJ)
- [Hic et nunc Github](https://github.com/hicetnunc2000/)

# hic et nunc
*here and now* 

### What is Hic Et Nunc?


[Hic et nunc](hicetnunc.xyz/) is a decentralized NFT marketplace built on the Tezos blockchain. It enables users to create, sell and interact with Tezos NFTs called OBJKTS. Each OBJKT holds a single artwork containing a 3d model, image, video, html snippet, glsl shader, etc. Hic et nunc lets creators limit how many digital versions of their work are in existence.

When a user uploads their content to hic et nunc, the actual file is uploaded to [IPFS](https://ipfs.io/), a decentralized storage network. The other metadata is stored in the Tezos blockchain. 

# Contact
Created by [@ianpetrarca](https://www.twitter.com/ianpetrarca) - tz1LobSdhfUqYpMojXWHQLJPhFLEzUEd9JAn
