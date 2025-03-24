// Inicialización de Three.js 
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Crear modelo 3D del cerebro (formas geométricas simples)
const cerebro = new THREE.Group();

const crearEtiqueta = (texto, x, y, z) => {
    const canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 64;
    const context = canvas.getContext('2d');
    context.font = '28px Arial';
    context.fillStyle = 'white';
    context.fillText(texto, 10, 40);
    const texture = new THREE.CanvasTexture(canvas);
    const spriteMaterial = new THREE.SpriteMaterial({ map: texture });
    const sprite = new THREE.Sprite(spriteMaterial);
    sprite.position.set(x, y + 1.5, z);
    sprite.scale.set(3, 1, 1);
    cerebro.add(sprite);
};

const nucleosRafeMesh = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshBasicMaterial({ color: 0x0000ff })
);
nucleosRafeMesh.position.set(-3, 0, 0);
cerebro.add(nucleosRafeMesh);
crearEtiqueta('Núcleos del Rafe', -3, 0, 0);

const vtaSnMesh = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);
vtaSnMesh.position.set(3, 0, 0);
cerebro.add(vtaSnMesh);
crearEtiqueta('VTA / Sustancia Negra', 3, 0, 0);

const cortezaPrefrontalMesh = new THREE.Mesh(
    new THREE.BoxGeometry(2, 2, 2),
    new THREE.MeshBasicMaterial({ color: 0xffff00 })
);
cortezaPrefrontalMesh.position.set(0, 3, 0);
cerebro.add(cortezaPrefrontalMesh);
crearEtiqueta('Corteza Prefrontal', 0, 3, 0);

const gangliosBasalesMesh = new THREE.Mesh(
    new THREE.BoxGeometry(2, 2, 2),
    new THREE.MeshBasicMaterial({ color: 0xff0000 })
);
gangliosBasalesMesh.position.set(0, -3, 0);
cerebro.add(gangliosBasalesMesh);
crearEtiqueta('Ganglios Basales', 0, -3, 0);

scene.add(cerebro);
camera.position.z = 10;

// Definir neurotransmisores (sistemas de partículas)
const serotoninaParticles = [];
const dopaminaParticles = [];
const gabaParticles = [];

// Parámetros de liberación
const params = {
    frecuenciaSerotonina: 1,
    cantidadSerotonina: 10,
    frecuenciaDopamina: 1,
    cantidadDopamina: 10,
    frecuenciaGaba: 1,
    cantidadGaba: 10
};

// Última liberación
let ultimaLiberacionSerotonina = 0;
let ultimaLiberacionDopamina = 0;
let ultimaLiberacionGaba = 0;

function liberarNeurotransmisor(particlesArray, region, color, cantidad) {
    for (let i = 0; i < cantidad; i++) {
        const particle = new THREE.SphereGeometry(0.1, 8, 8);
        const particleMesh = new THREE.Mesh(particle, new THREE.MeshBasicMaterial({ color: color }));
        particleMesh.position.copy(region.position);
        scene.add(particleMesh);
        particlesArray.push(particleMesh);
    }
}

function simularDifusion(particlesArray) {
    particlesArray.forEach(particle => {
        particle.position.x += (Math.random() - 0.5) * 0.1;
        particle.position.y += (Math.random() - 0.5) * 0.1;
        particle.position.z += (Math.random() - 0.5) * 0.1;
    });
}

// Integrar dat.GUI para controles interactivos
const gui = new dat.GUI();
const serotoninaFolder = gui.addFolder('Serotonina');
serotoninaFolder.add(params, 'frecuenciaSerotonina', 0.1, 10, 0.1);
serotoninaFolder.add(params, 'cantidadSerotonina', 1, 100, 1);

const dopaminaFolder = gui.addFolder('Dopamina');
dopaminaFolder.add(params, 'frecuenciaDopamina', 0.1, 10, 0.1);
dopaminaFolder.add(params, 'cantidadDopamina', 1, 100, 1);

gabaFolder = gui.addFolder('GABA');
gabaFolder.add(params, 'frecuenciaGaba', 0.1, 10, 0.1);
gabaFolder.add(params, 'cantidadGaba', 1, 100, 1);

gui.close();

// Bucle principal
function animate() {
    requestAnimationFrame(animate);

    const now = performance.now() / 1000;

    if (now - ultimaLiberacionSerotonina >= 1 / params.frecuenciaSerotonina) {
        liberarNeurotransmisor(serotoninaParticles, nucleosRafeMesh, 0x00ffff, params.cantidadSerotonina);
        ultimaLiberacionSerotonina = now;
    }

    if (now - ultimaLiberacionDopamina >= 1 / params.frecuenciaDopamina) {
        liberarNeurotransmisor(dopaminaParticles, vtaSnMesh, 0xff00ff, params.cantidadDopamina);
        ultimaLiberacionDopamina = now;
    }

    if (now - ultimaLiberacionGaba >= 1 / params.frecuenciaGaba) {
        liberarNeurotransmisor(gabaParticles, gangliosBasalesMesh, 0x808080, params.cantidadGaba);
        ultimaLiberacionGaba = now;
    }

    simularDifusion(serotoninaParticles);
    simularDifusion(dopaminaParticles);
    simularDifusion(gabaParticles);

    renderer.render(scene, camera);
}

animate();

const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.update();
