# Image Sharpness Map Generator 

A web-based tool that analyzes image sharpness by generating visual heatmaps to highlight sharp vs blurry regions in photographs.

## Features 
- **Heatmap Visualization**: Color-coded sharpness mapping (blue = blurry, red = sharp)
- **Real-time Processing**: Analyzes images client-side with JavaScript
- **Laplacian Edge Detection**: Uses convolution kernels to detect sharp edges
- **Responsive Design**: Works on desktop and mobile devices


## How It Works 
1. Upload an image or use the sample image
2. The tool converts the image to grayscale
3. Applies a Laplacian filter to detect edges (sharpness)
4. Normalizes the results and generates a heatmap overlay
5. Displays side-by-side comparison of original and sharpness map

##Installation 
1. Simply open `index.html` in any modern browser
