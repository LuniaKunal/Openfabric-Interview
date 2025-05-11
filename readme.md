# AI Creative

AI Creative is an end-to-end pipeline for generating 3D models from text prompts, leveraging cutting-edge AI technologies including text-to-image generation and image-to-3D model transformation.


## Features

- **Text-to-Image Generation**: Convert descriptive text prompts into detailed, high-quality images
- **Text Prompt Enhancement**: Automatically enhance user prompts for better results using LLM
- **Image-to-3D Conversion**: Transform 2D images into textured 3D models in GLB format
- **Video Preview**: Generate video previews of the 3D models for quick visualization
- **User-friendly Interface**: Simple web UI for interacting with the application
- **Openfabric Integration**: Leverages Openfabric's powerful AI applications ecosystem

## System Requirements

- Python 3.10+
- 8GB RAM (16GB+ recommended for larger models)
- Modern CPU (GPU recommended for improved performance)
- 1GB free disk space for the application and generated assets
- 13GB free disk space for the LLM model (if downloaded)
- Internet connection (for Openfabric API access)
- Operating Systems: Windows 10/11, macOS, Linux

## Default Model Used

- The LLM used is the `google/gemma-3-1b-it` model, which is a lightweight version of the Llama-3.2 model. It is designed to run on consumer-grade hardware with 8GB of RAM. The model is capable of enhancing text prompts for better image generation results.

- This is a gated model; you would need your huggingface token to access it. You can request access to the model [here](https://huggingface.co/google/gemma-3-1b-it).

- The model is downloaded using the `download_model.sh` script, which uses the `huggingface-cli` to authenticate and download the model files. Set your Hugging Face token in the `.env` file before running the script.

- If you don't want to add your Huggingface token, the application will fall back to using non-gated models.

```
    fallback_models = [
        "google/gemma-3-1b-it",
        # Can add different models
    ]
```


## Installation

### 1. Clone the repository

```bash
git clone https://github.com/LuniaKunal/Openfabric-Interview.git
```

### 2. Create a Python virtual environment

```bash
# Using conda
conda create -n text-image-3d python=3.10
conda activate text-image-3d

```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Download the LLM model (optional but **recommended**)

Model size is approximately 13GB.
To use the local LLM for prompt enhancement:

```bash
bash download_model.sh
```

### 5. Set up environment variables

Create a `.env` file in the project root. Check `.env.example` for the required variables.

## Usage

### Starting the LLM Service

Start the local LLM service first:

```bash
python app/llm/service.py
```

Wait for the model to load (you should see "LLM initialized successfully" in the console).

### Running the UI Application

In a new terminal window:

```bash
python app/ui/app.py
```

This will launch the web interface, accessible at [http://localhost:7860](http://localhost:7860).

### Using the Application

1. **Enter a prompt**: Enter a descriptive text prompt for the 3D model you want to create
2. **Generate**: Click the "Generate" button to start the creation process
3. **View results**: The application will display:
   - The original prompt
   - The enhanced prompt (if LLM enhancement worked)
   - The generated image
   - The 3D model viewer with the created model
   - Download links for both image and 3D model

<img width="1470" alt="Image Generation" src="assets\image-gen.png" />

Here's a screenshot of the application in action, showing the input prompt, generated image, and the expanded prompt. The total time taken for the generation process is also displayed.

<img width="1470" alt="Model Viewer" src="assets\Chair.png" />

_Model Viewer_

<img width="1470" alt="Image Gallery" src="assets\Chair-3D.png" />

_Image Gallery_

<img width="1470" alt="Model Gallery" src="assets\amror-3D.png" />

_Model Gallery_

## Memory Implementation

AI Creative implements a file-based persistence system for storing generated assets and their metadata. This allows the application to maintain a history of creations and reload them even after application restarts.

### Image Storage

When a text-to-image generation is completed:

1. **Image Files**: Generated images are saved in the `app/data/images/` directory as PNG files
2. **Metadata Files**: For each image, a corresponding JSON metadata file is created with the same base name
3. **Metadata Content**: The metadata includes:
   - Original prompt
   - Enhanced prompt
   - Timestamp
   - Generation parameters
   - Result IDs from Openfabric
   - Download status

Example of image metadata:

```json
{
  "id": "28def601-6e40-4924-834f-9327446142e4",
  "timestamp": 1746984813,
  "prompt": "A weathered, crimson-red rabit, perched atop a crumbling stone altar, dominates the frame, its golden eyes gleaming with a mixture of defiance and ancient wisdom. Sunlight streams through a shattered stained-glass window, casting a mosaic of violet and emerald hues across the dusty floor, illuminating swirling motes of gold dust and highlighting the intricate details of the altar\u2019s carvings. The scene is rendered in a dramatic, chiaroscuro style reminiscent of Caravaggio, evoking a sense of melancholic grandeur and a palpable sense of forgotten power. The rabit\u2019s fur is a patchwork of deep browns and blacks, subtly textured with a rough, almost granular appearance, and the altar itself is constructed from aged, moss-covered granite.",
  "parameters": {},
  "result_id": "data_blob_31f62264a315fb69d3018eec1f5950e78dbecfe90558615026321b2e37498064/executions/4feb83f688be421c9ae21bd9597357ca",
  "type": "image",
  "needs_download": false,
  "base_name": "Rabit_with_a_cr",
  "original_prompt": "Rabit with a crown on his head",
  "expanded_prompt": "A weathered, crimson-red rabit, perched atop a crumbling stone altar, dominates the frame, its golden eyes gleaming with a mixture of defiance and ancient wisdom. Sunlight streams through a shattered stained-glass window, casting a mosaic of violet and emerald hues across the dusty floor, illuminating swirling motes of gold dust and highlighting the intricate details of the altar\u2019s carvings. The scene is rendered in a dramatic, chiaroscuro style reminiscent of Caravaggio, evoking a sense of melancholic grandeur and a palpable sense of forgotten power. The rabit\u2019s fur is a patchwork of deep browns and blacks, subtly textured with a rough, almost granular appearance, and the altar itself is constructed from aged, moss-covered granite."
}
```

### Model Storage

For 3D model generation:

1. **Model Files**: Generated 3D models are saved in the `app/data/models/` directory as GLB files
2. **Metadata Files**: Each model has an associated JSON metadata file
3. **Metadata Content**: The model metadata includes:
   - Source image reference
   - Model format (always GLB for better compatibility)
   - Timestamp
   - Generation parameters
   - Video preview path (if available)

### Persistence and Loading

- **On-disk Persistence**: All assets and metadata are written to disk immediately after generation
- **Directory Structure**: The application maintains a clean directory structure with separate folders for images and models
- **Auto-discovery**: When the application starts, it scans the data directories to find and load existing assets
- **UI Integration**: Previously generated assets are accessible through the galleries in the UI
- **Download Management**: For remotely stored assets, the application tracks download status and can retrieve them when needed

### Blob Storage Integration

The application can also handle remote assets stored in Openfabric's blob storage:

1. When an image or model is generated, it may initially exist only as a reference to a remote blob
2. The application downloads these blobs as needed and stores them locally
3. The metadata files are updated to reflect the local storage path once downloaded

I couldn't fully implement the instructed memory specs in openfabric_guide.md, but I could if I had more time. The current implementation focuses on a simple file-based approach rather than the more sophisticated database solution outlined in the guide.

## Architecture

AI Creative consists of several key components:

- **Core Pipeline**: Orchestrates the end-to-end creative workflow
- **LLM Service**: Enhances text prompts using a local language model
- **Text-to-Image Service**: Generates images from text (via Openfabric)
- **Image-to-3D Service**: Creates 3D models from images (via Openfabric)
- **Web UI**: Gradio-based user interface

The application uses Openfabric's platform for the compute-intensive image and 3D model generation tasks, while running a lightweight LLM locally for prompt enhancement.

## Logging

AI Creative maintains detailed logs in the following locations:

- `app/core/openfabric_service.log`: Logs for Openfabric service operations
- `app/llm/llm_service.log`: Logs for the local LLM service

## Troubleshooting

### Common Issues

- **LLM Service not starting**: Ensure you have sufficient RAM for the model, and check that the model download was completed successfully
- **Connection errors**: Verify your internet connection and check that your Openfabric app IDs are correct
- **Generation timeout**: Image or 3D model generation may take time, depending on complexity and service load
- **"No module found" errors**: Ensure all dependencies are installed and you're using the correct Python environment
