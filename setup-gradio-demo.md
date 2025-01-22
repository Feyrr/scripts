# CompanyA Food PoC
---
This is a Gradio demo for a PoC for CompanyB's potential deal with CompanyA Food.

The thing is this: CompanyA wants to have some food photo but don't want to spend too much getting them and is resorting to AI to help.

## How it works
1. Get an extremely detailed image description of the food photo with GPT-4v.
2. Use GPT to perform an image edit by making small modifications to the image description obtained from 1.
3. Make use of IP-Adapter or more specifically InstantStyle.
4. During inference, use a good weightage for image and text conditions by adjusting the `scale` parameter. If required, further perform semantic moditication to the image embeddings using negative and positive content concepts/embeddings.

## Assumptions
1. The name of the food is known.
2. A food photo is provided.

## Usage
```bash
cd gradio_demo_CompanyA_food_variation

# Download IP adapter XL model from Huggingface Hub to sdxl_models/
# Note: Not the ViT-H nor Plus version!!!
wget https://huggingface.co/h94/IP-Adapter/resolve/main/sdxl_models/ip-adapter_sdxl.bin -O sdxl_models/ip-adapter_sdxl.bin

# Create Python virtual environment and install Python dependencies
python -m virtualenv venv
source venv/bin/activate
pip install -r requirements.txt

# Set OPENAI_API_KEY environment variable
export OPENAI_API_KEY=...

python main_CompanyA_food_ipadapterxl.py
```

## Dockerizing the Application
Follow these steps to dockerize the application:

**Clone the Repository and Download the Model** After cloning this repository, download the IP Adapter XL model from the Hugging Face Hub and place the model file in the `sdxl_models/` folder:
> ❗ **Note: Not the ViT-H nor Plus version!!!**

```
wget https://huggingface.co/h94/IP-Adapter/resolve/main/sdxl_models/ip-adapter_sdxl.bin -O sdxl_models/ip-adapter_sdxl.bin
```
**Create a `.env` File** Set up a `.env` file to store your API keys. Create the file in the root of your project directory and include the following lines, replacing the placeholders with your actual keys:
```
OPENAI_API_KEY=insert_openai_secret_here
PRODIA_API_KEY=insert_prodia_secret_here
```

**Build the Docker Image** Use Docker Compose to build the image and start the application. Run the following command in your terminal:
```
docker compose down && docker compose up --build --remove-orphans -d
```
This command stops any running containers, builds the image, and starts the application in detached mode.

**Monitor the Build Process** The build process may take several minutes. You can check the logs to monitor the status of the application. Use the following command, replacing `container_name` with the name of your container:
```
docker logs -f --tail 10 container_name
```
Additionally, you can monitor GPU usage while the application is running by executing:
```
watch nvidia-smi
```
When you see the following logs in your container logs, it indicates that the application is ready to serve:
```
csharp
Copy code
CUDA Version 12.6.2

==========
== CUDA ==
==========

By pulling and using the container, you accept the terms and conditions of this license:
This container image and its contents are governed by the NVIDIA Deep Learning Container License.

https://developer.nvidia.com/ngc/nvidia-deep-learning-container-license
A copy of this license is made available in this container at /NGC-DL-CONTAINER-LICENSE for your convenience.

Settings -> Mode=base, Device=cpu, Torchscript=disabled

Running on local URL:  http://0.0.0.0:80

To create a public link, set `share=True` in `launch()`.
```
