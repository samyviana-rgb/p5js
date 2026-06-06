/*
  Experimento Glitch Momentum: Câmera, Ruído Neural e Pixel Sorting
  Inspirado nas teorias de Rosa Menkman e na estética do acidente tecnológico.
*/

let video;
let thresholdSlider;

function setup() {
  // Criamos uma tela que dialogue com a proporção padrão de vídeo
  createCanvas(640, 480);
  
  // Inicializa a captura da câmera (o fluxo procedural)
  video = createCapture(VIDEO);
  video.size(640, 480);
  video.hide(); // Ocultamos o elemento HTML nativo para controlar a renderização no canvas
  
  // Slider para controlar o "limiar neural" do glitch em tempo real
  thresholdSlider = createSlider(0, 255, 100);
  thresholdSlider.position(10, height + 10);
  
  pixelDensity(1);
  background(0);
}

function draw() {
  // Carrega os pixels do vídeo capturado
  video.loadPixels();
  
  // Carrega os pixels do canvas onde faremos a disfiguração
  loadPixels();
  
  let threshold = thresholdSlider.value();
  
  // Percorre a matriz de pixels (interleaved RGBA)
  for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
      let loc = (x + y * width) * 4;
      
      // Extrai os valores de cor da câmera
      let r = video.pixels[loc];
      let g = video.pixels[loc+1];
      let b = video.pixels[loc+2];
      
      // Calcula o brilho/luminância (similar aos processos de Downsampling Y'CbCr)
      let brightness = (r + g + b) / 3;
      
      // Se o brilho quebrar o script estabelecido (threshold), aplicamos o Pixel Sorting / Ruído
      if (brightness > threshold && x > 0 && random(100) > 15) {
        // "Datamoshing" conceitual: pega o pixel anterior da linha para gerar o efeito de arrasto
        let prevLoc = (x - 1 + y * width) * 4;
        pixels[loc]   = pixels[prevLoc];
        pixels[loc+1] = pixels[prevLoc+1];
        pixels[loc+2] = pixels[prevLoc+2];
        pixels[loc+3] = 255;
      } else {
        // Transmissão normal do canal de dados
        pixels[loc]   = r;
        pixels[loc+1] = g;
        pixels[loc+2] = b;
        pixels[loc+3] = 255;
      }
      
      // Injeção de Ruído Aleatório Estático (Spike de Voltagem / Glitch Puro)
      if (random(10000) > 9995) {
        pixels[loc]   = random(255);
        pixels[loc+1] = random(255);
        pixels[loc+2] = random(255);
      }
    }
  }
  
  // Atualiza o canvas com a nova superfície disfigurada
  updatePixels();
  
  // Intervenção Gráfica: Simulação de quebra de sincronia vertical/horizontal (Combing Artifacts)
  if (frameCount % 30 < 5) {
    let sliceY = int(random(height));
    let sliceHeight = int(random(5, 30));
    let hOffset = int(random(-50, 50));
    copy(this, 0, sliceY, width, sliceHeight, hOffset, sliceY + int(random(-5, 5)), width, sliceHeight);
  }
  
  // Texto informativo sobre o estado do sistema (Aparato Crítico)
  fill(0, 255, 0);
  noStroke();
  fontFamily = 'monospace';
  text("GLITCH MOMENTUM: ACTIVE", 15, 20);
  text("THRESHOLD: " + threshold, 15, 35);
}
