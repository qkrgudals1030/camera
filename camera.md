### 실습화면

![image](https://github.com/qkrgudals1030/camera/assets/50895124/087a4f53-0c30-4724-bb6e-a931b8af622f)

### 실습코드 


```
import * as THREE from '../build/three.module.js';
import {OrbitControls} from '../examples/jsm/controls/OrbitControls.js';
import {RectAreaLightUniformsLib} from '../examples/jsm/lights/RectAreaLightUniformsLib.js';
import {RectAreaLightHelper} from '../examples/jsm/helpers/RectAreaLightHelper.js';

class App {
   constructor() {
      const divContainer = document.querySelector('#webgl-container');
      this._divContainer = divContainer;

      const renderer = new THREE.WebGLRenderer({antialias: true});
      renderer.setPixelRatio(window.devicePixelRatio);
      divContainer.appendChild(renderer.domElement);
      this._renderer = renderer;

      const scene = new THREE.Scene();
      this._scene = scene;

      this._setupCamera();
      this._setupLight();
      this._setupModel();
      this._setupControls();

      window.onresize = this.resize.bind(this);
      this.resize();

      requestAnimationFrame(this.render.bind(this));
   }

   _setupControls() {
      new OrbitControls(this._camera, this._divContainer);
   }

   _setupCamera() {
      const width = this._divContainer.clientWidth;
      const height = this._divContainer.clientHeight;

  
      const camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 100);

      // OrthographicCamera(수평축 왼쪽에 대한 좌표값, 수평축 오른쪽에 대한 좌표값, 수직축 위쪽에 대한 좌표값, 수직축 아래쪽에 대한 좌표값, 카메라로부터의 거리 앞쪽, 카메라로부터의 거리 뒤쪽)
      // aspect : 렌더링 결과가 표시되는 DOM 요소 크기에 대한 종횡비를 적용
      /* const aspect = window.innerWidth / window.innerHeight;
      const camera = new THREE.OrthographicCamera(aspect * -1, aspect * 1, 1, -1, 0.1, 100);
      camera.zoom = 0.15; */

      //camera.position.z = 2;
   
      camera.position.set(7, 7, 0);

      camera.lookAt(0, 0, 0);
      this._camera = camera;
   }

   _setupLight() {
      RectAreaLightUniformsLib.init();
      const light = new THREE.RectAreaLight(0xffffff, 10, 6, 1);
      light.position.set(0, 5, 0);
      light.rotation.x = THREE.MathUtils.degToRad(-90);

      const helper = new RectAreaLightHelper(light);
      this._scene.add(helper);
      this._lightHelper = helper;

      this._scene.add(light);
      this._light = light;
   }

   _setupModel() {
      const groundGeometry = new THREE.PlaneGeometry(10, 10);
      const groundMaterial = new THREE.MeshStandardMaterial({
         color: '#2c3e50',
         roughness: 0.5,
         metalness: 0.5,
         side: THREE.DoubleSide,
      });

      const ground = new THREE.Mesh(groundGeometry, groundMaterial);
      ground.rotation.x = THREE.MathUtils.degToRad(-90);
      this._scene.add(ground);

      const bigSphereGeometry = new THREE.SphereGeometry(1.5, 64, 64, 0, Math.PI);
      const bigSphereMaterial = new THREE.MeshStandardMaterial({
         color: '#3498db',
         roughness: 0.1,
         metalness: 0.2,
      });

      const bigSphere = new THREE.Mesh(bigSphereGeometry, bigSphereMaterial);
      bigSphere.rotation.x = THREE.MathUtils.degToRad(-90);
      this._scene.add(bigSphere);

      const torusGeometry = new THREE.TorusGeometry(0.4, 0.1, 32, 32);
      const torusMaterial = new THREE.MeshStandardMaterial({
         color: '#27ae60',
         roughness: 0.5,
         metalness: 0.9,
      });

      for (let i = 0; i < 8; i++) {
         const torusPivot = new THREE.Object3D();
         const torus = new THREE.Mesh(torusGeometry, torusMaterial);
         torusPivot.rotation.y = THREE.MathUtils.degToRad(45 * i);
         torus.position.set(3, 0.5, 0);
         torusPivot.add(torus);
         this._scene.add(torusPivot);
      }

      const smallSphereGeometry = new THREE.SphereGeometry(0.3, 32, 32);
      const smallSphereMaterial = new THREE.MeshStandardMaterial({
         color: '#e74c3c',
         roughness: 0.2,
         metalness: 0.5,
      });

      const smallSpherePivot = new THREE.Object3D();
      const smallSphere = new THREE.Mesh(smallSphereGeometry, smallSphereMaterial);
      smallSpherePivot.add(smallSphere);
      smallSpherePivot.name = 'smallSpherePivot';
      smallSphere.position.set(3, 0.5, 0);
      this._scene.add(smallSpherePivot);

      const targetPivot = new THREE.Object3D();
      const target = new THREE.Object3D();
      targetPivot.add(target);
      targetPivot.name = 'targetPivot';
      target.position.set(3, 0.5, 0);
      this._scene.add(targetPivot);
   }

   resize() {
      const width = this._divContainer.clientWidth;
      const height = this._divContainer.clientHeight;
      const aspect = width / height;

      if (this._camera instanceof THREE.PerspectiveCamera) {
         // PerspectiveCamera
         this._camera.aspect = aspect;
      } else {
         // OrthographicCamera
         this._camera.left = aspect * -1;
         this._camera.right = aspect * 1;
      }

      this._camera.updateProjectionMatrix();

      this._renderer.setSize(width, height);
   }

   render(time) {
      this._renderer.render(this._scene, this._camera);
      this.update(time);
      requestAnimationFrame(this.render.bind(this));
   }

   update(time) {
      time *= 0.001; // second unit

      const smallSpherePivot = this._scene.getObjectByName('smallSpherePivot');
      if (smallSpherePivot) {
         smallSpherePivot.rotation.y = THREE.MathUtils.degToRad(time * 50);

         const smallSphere = smallSpherePivot.children[0];
         smallSphere.getWorldPosition(this._camera.position);

         const targetPivot = this._scene.getObjectByName('targetPivot');
         if (targetPivot) {
            targetPivot.rotation.y = THREE.MathUtils.degToRad(time * 50 + 10);

            const target = targetPivot.children[0];
            const pt = new THREE.Vector3();

            target.getWorldPosition(pt);

            this._camera.lookAt(pt);
         }

         if (this._light.target) {
            const smallSphere = smallSpherePivot.children[0];
            smallSphere.getWorldPosition(this._light.target.position);

            if (this._lightHelper) {
               this._lightHelper.update();
            }
         }
      }

   }
}

window.onload = function() {
   new App();
}

```


### 소감

orthographiccamera는 어제 실습한 light의 예제의 실습화면으로 3인칭이 였다고 한다면 오늘 퀴즈시험으로 수행한 perspectivecamera의 경우에는 작은 원의 1인칭시점으로 카메라가 작동하는 실습이였습니다. 이렇듯 다양한 사물들의 여러 상호작용하는 모습을 보며 실습을 해보니 실제 게임에서 보던 오브젝트들을 
만드는 것에는 다양한 상호작용이 적용될 수 있다고 생각이 되었고 그런 게임에서 제작한 오브젝트를 만드는 것처럼 복잡한 상호작용을 하는 오브젝트를 생성해 볼 수 있도록 열심히 공부해 보겠습니다. 그래픽스 화이팅!!!




