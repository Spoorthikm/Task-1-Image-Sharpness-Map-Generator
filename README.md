# Task-1-Image-Sharpness-Map-Generator 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Image Sharpness Map Generator</title>
<link href="https://fonts.googleapis.com/css2?family=Inter&family=Material+Icons" rel="stylesheet" />
<style>
  /* Reset and base */
  *, *::before, *::after {
    box-sizing: border-box;
  }
  body {
    margin: 0; padding: 0;
    font-family: 'Inter', sans-serif;
    background: linear-gradient(135deg, #1e293b, #0f172a);
    color: #f1f5f9;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }
  header {
    padding: 1.5rem 2rem;
    background: rgba(30, 41, 59, 0.75);
    backdrop-filter: blur(12px);
    box-shadow: 0 1px 12px rgba(0,0,0,0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.8rem;
    font-weight: 700;
    user-select: none;
    position: sticky;
    top: 0;
    z-index: 100;
  }
  main {
    flex-grow: 1;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 32px;
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
    width: 100%;
  }
  @media (max-width: 900px) {
    main {
      grid-template-columns: 1fr;
      padding: 1rem 1rem 3rem;
    }
  }
  .card {
    background: rgba(30,41,59,0.6);
    border-radius: 16px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.4);
    backdrop-filter: blur(18px);
    padding: 24px;
    display: flex;
    flex-direction: column;
    align-items: center;
    position: relative;
    min-height: 520px;
  }
  h2 {
    margin: 0 0 1rem 0;
    font-weight: 600;
    font-size: 1.5rem;
    user-select: none;
  }
  .upload-section {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  input[type="file"] {
    margin-top: 0.5rem;
    padding: 8px 12px;
    background: transparent;
    border: 1.5px solid #f59e0b;
    border-radius: 8px;
    color: #f59e0b;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.3s ease, color 0.3s ease;
  }
  input[type="file"]::-webkit-file-upload-button {
    cursor: pointer;
    background: transparent;
    border: none;
    color: #f59e0b;
    font-weight: 600;
    padding: 0;
  }
  input[type="file"]:hover {
    background-color: #f59e0b;
    color: #0f172a;
  }
  .image-container {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    max-width: 100%;
    max-height: 420px;
    overflow: hidden;
  }
  img#sourceImage {
    max-width: 100%;
    max-height: 420px;
    border-radius: 12px;
    box-shadow: 0 6px 15px rgba(245, 158, 11, 0.6);
    user-select: none;
    image-rendering: auto;
    object-fit: contain;
  }
  canvas#sharpnessCanvas {
    border-radius: 12px;
    max-width: 100%;
    max-height: 420px;
    box-shadow: 0 6px 15px rgba(245, 158, 11, 0.6);
    image-rendering: pixelated;
  }
  .footer-note {
    font-size: 0.9rem;
    color: #fbbf24;
    margin-top: 0.5rem;
    text-align: center;
    user-select: none;
  }
  .material-icons {
    vertical-align: middle;
    color: #f59e0b;
    font-size: 1.6rem;
  }
</style>
</head>
<body>
<header>
  <span class="material-icons" aria-hidden="true">photo_camera</span>&nbsp;
  Image Sharpness Map Generator
</header>
<main>
  <section class="card" aria-label="Original Image">
    <h2>Original Image</h2>
    <div class="upload-section">
      <label for="imageUpload" style="cursor:pointer; font-weight: 600; color: #f59e0b;">Choose Image</label>
      <input type="file" id="imageUpload" accept="image/*" aria-describedby="uploadNote" />
      <div id="uploadNote" class="footer-note">Select an image to analyze sharpness</div>
    </div>
    <div class="image-container" style="margin-top: 1rem;">
      <img id="sourceImage" src="https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/1d768f9a-a40e-4f0b-976c-8cf6a5519ebc.png" alt="Placeholder image showing instructions to upload or select an image" />
    </div>
  </section>
  <section class="card" aria-label="Sharpness Heatmap">
    <h2>Sharpness Heatmap</h2>
    <canvas id="sharpnessCanvas" width="600" height="400" role="img" aria-label="Sharpness heatmap visualization"></canvas>
    <div class="footer-note" style="margin-top: 0.8rem;">Heatmap shows sharper (bright areas) vs blurrier (dark areas) regions.</div>
  </section>
</main>

<script>
  // Clamp number to a range
  function clamp(value, min, max) {
    return Math.min(Math.max(value, min), max);
  }
  // Convert RGB to gray luminance
  function rgbToGray(r, g, b) {
    return 0.2126 * r + 0.7152 * g + 0.0722 * b;
  }
  // 3x3 convolution kernel application to 1-channel data
  function convolve3x3(data, width, height, kernel) {
    const output = new Float32Array(width * height);
    for(let y = 1; y < height - 1; y++) {
      for(let x = 1; x < width - 1; x++) {
        let sum = 0;
        for(let ky = -1; ky <= 1; ky++) {
          for(let kx = -1; kx <= 1; kx++) {
            const px = x + kx;
            const py = y + ky;
            const weight = kernel[(ky + 1) * 3 + (kx + 1)];
            sum += data[py * width + px] * weight;
          }
        }
        output[y * width + x] = sum;
      }
    }
    return output;
  }
  // Normalize float data array to 0-255 Uint8 array
  function normalizeTo255(data) {
    let minVal = Infinity;
    let maxVal = -Infinity;
    for(const val of data){
      if(val < minVal) minVal = val;
      if(val > maxVal) maxVal = val;
    }
    const range = maxVal - minVal || 1;
    const normArray = new Uint8ClampedArray(data.length);
    for(let i = 0; i < data.length; i++) {
      const normalized = (data[i] - minVal) / range;
      normArray[i] = Math.round(clamp(normalized, 0, 1) * 255);
    }
    return normArray;
  }
  // Convert a value 0-255 to a color on blue->red heatmap gradient
  function heatmapColor(value) {
    const v = value / 255;
    const hue = (1 - v) * 240; // 240 = blue, 0 = red
    return hsvToRgb(hue, 1, 1);
  }
  // HSV to RGB color conversion
  function hsvToRgb(h, s, v) {
    const c = v * s;
    const hp = h / 60;
    const x = c * (1 - Math.abs((hp % 2) - 1));
    let r = 0, g = 0, b = 0;
    if (hp >= 0 && hp < 1) [r, g, b] = [c, x, 0];
    else if (hp >= 1 && hp < 2) [r, g, b] = [x, c, 0];
    else if (hp >= 2 && hp < 3) [r, g, b] = [0, c, x];
    else if (hp >= 3 && hp < 4) [r, g, b] = [0, x, c];
    else if (hp >= 4 && hp < 5) [r, g, b] = [x, 0, c];
    else if (hp >= 5 && hp < 6) [r, g, b] = [c, 0, x];
    const m = v - c;
    return {
      r: Math.round((r + m) * 255),
      g: Math.round((g + m) * 255),
      b: Math.round((b + m) * 255)
    };
  }

  // Main elements references
  const imageUpload = document.getElementById('imageUpload');
  const sourceImage = document.getElementById('sourceImage');
  const sharpnessCanvas = document.getElementById('sharpnessCanvas');
  const ctxHeatmap = sharpnessCanvas.getContext('2d');

  // Create sharpness heatmap from image element (uses resized max 600x400)
  function createSharpnessMap(image) {
    const maxWidth = 600;
    const maxHeight = 400;
    let iw = image.naturalWidth;
    let ih = image.naturalHeight;
    let scale = 1;
    if(iw > maxWidth || ih > maxHeight) {
      scale = Math.min(maxWidth/iw, maxHeight/ih);
      iw = Math.floor(iw*scale);
      ih = Math.floor(ih*scale);
    }

    // Draw image to offscreen canvas
    const offCanvas = document.createElement('canvas');
    offCanvas.width = iw;
    offCanvas.height = ih;
    const offCtx = offCanvas.getContext('2d');
    offCtx.clearRect(0,0,iw,ih);
    offCtx.drawImage(image, 0, 0, iw, ih);

    // Get image data
    const imgData = offCtx.getImageData(0, 0, iw, ih);
    const pixels = imgData.data;

    // Convert to grayscale luminance
    const grayData = new Float32Array(iw*ih);
    for(let i = 0, j = 0; i < pixels.length; i += 4, j++) {
      grayData[j] = rgbToGray(pixels[i], pixels[i+1], pixels[i+2]);
    }

    // Laplacian kernel
    const laplacianKernel = [
       0,  1,  0,
       1, -4,  1,
       0,  1,  0
    ];
    const laplacianData = convolve3x3(grayData, iw, ih, laplacianKernel);

    // Absolute values for magnitude of edges
    for(let i = 0; i < laplacianData.length; i++) {
      laplacianData[i] = Math.abs(laplacianData[i]);
    }

    // Normalize 0-255
    const normSharpness = normalizeTo255(laplacianData);

    // Create heatmap image data
    const heatmapImageData = ctxHeatmap.createImageData(iw, ih);
    for(let i = 0; i < normSharpness.length; i++) {
      const {r, g, b} = heatmapColor(normSharpness[i]);
      heatmapImageData.data[i*4    ] = r;
      heatmapImageData.data[i*4 + 1] = g;
      heatmapImageData.data[i*4 + 2] = b;
      heatmapImageData.data[i*4 + 3] = 230; // Alpha (semi-transparent)
    }

    // Resize canvas and draw heatmap
    sharpnessCanvas.width = iw;
    sharpnessCanvas.height = ih;
    ctxHeatmap.putImageData(heatmapImageData, 0, 0);
  }

  // File input change handler to load image and trigger heatmap
  imageUpload.addEventListener('change', (event) => {
    const file = event.target.files[0];
    if(!file) return;

    const url = URL.createObjectURL(file);
    sourceImage.src = url;
    sourceImage.alt = file.name + " uploaded image";
  });

  // When image finishes loading (original or uploaded), generate heatmap
  sourceImage.addEventListener('load', () => {
    if(!sourceImage.naturalWidth || !sourceImage.naturalHeight) return;
    createSharpnessMap(sourceImage);
  });

  // On page load, generate heatmap for placeholder image if loaded
  window.addEventListener('load', () => {
    if(sourceImage.complete && sourceImage.naturalWidth && sourceImage.naturalHeight) {
      createSharpnessMap(sourceImage);
    }
  });
</script>
</body>
</html>

