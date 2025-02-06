# Serve 671B DeepSeek-R1 on your locall!
Here, we provide a comprehensive guide on serving the 671B DeepSeek-R1 model on a local GPU setup from start to finish.
All you need are 2 A100 GPUs to get started.
Follow the steps below to serve the 671B DeepSeek-R1 model.
We hope to make it easier for users to try by consolidating scattered guides into a single, unified resource.

## 0. Build Docker Environment
You need to use the following command to build the Docker image that will be used as the DeepSeek model running environment:
```bash
cd docker
docker build -t ${IMAGE_NAME} .
```

Then, you can make a Docker container using the following command:
```bash
docker run -it -d --name ${CONTAINER_NAME} --gpus all --shm-size=2g -v ${PATH_TO_BE_MOUNTED}:${MOUNT_PATH} -p 7777:8080 ${IMAGE_NAME}
docker exec -it ${CONTAINER_NAME} /bin/bash
```
<br>



## 1. [Build llama.cpp](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md)
First, once you are in the docker container, you need to build llama.cpp.
```bash
# Install libraries
apt-get update
apt-get install build-essential cmake curl libcurl4-openssl-dev -y

# Build llama.cpp
git clone https://github.com/ggerganov/llama.cpp
cmake llama.cpp -B llama.cpp/build -DBUILD_SHARED_LIBS=OFF -DGGML_CUDA=ON -DLLAMA_CURL=ON
cmake --build llama.cpp/build --config Release -j --clean-first --target llama-quantize llama-cli llama-gguf-split llama-server
cp llama.cpp/build/bin/llama-* llama.cpp
```
<br>

## 2. Download DeepSeek Models
Then, you need to download the model locally using the below python script.
```python
# You can set your own model path at the 'local_dir' option. This is just an example.
from huggingface_hub import snapshot_download
snapshot_download(
  repo_id = "unsloth/DeepSeek-R1-GGUF",
  local_dir = "DeepSeek-R1-GGUF",
  allow_patterns = ["*UD-IQ1_S*"],
)
```
You can choose quantized model types using the following patterns:
* `UD-IQ1_S`: 1.58bit
* `UD-IQ1_M`: 1.73bit
* `UD-IQ2_XXS`: 2.22bit
* `UD-Q2_K_XL`: 2.51bit
<br><br>


## 3. Merging Split .gguf Models (Optional)
*This process is not mandatory! You can skip this process.*<br>
Here, we can merge split models into a single `.gguf` model.
When merging GGUF files, you only need to provide the name of the first file and the rest of the files will be automatically found and merged.
```bash
./llama.cpp/llama-gguf-split --merge DeepSeek-R1-GGUF/DeepSeek-R1-UD-IQ1_S/DeepSeek-R1-UD-IQ1_S-00001-of-00003.gguf DeepSeek-R1-GGUF/DeepSeek-R1-UD-IQ1_S/DeepSeek-R1-UD-IQ1_S-merged.gguf
```
<br>



## 4. Inference
Here, we provide guides for inference using the CLI and inference using the web UI.

---
### 4.1. Inference via CLI
When you laod the model, you can set only the first `.gguf` model file.
Then, llama.cpp will automatically find the other files.
```bash
# You can set your own model path at the '--model' option. This is just an example.
CUDA_VISIBLE_DEVICES=0,1 ./llama.cpp/llama-cli \
    --model DeepSeek-R1-GGUF/DeepSeek-R1-UD-IQ1_S/DeepSeek-R1-UD-IQ1_S-00001-of-00003.gguf \
    --cache-type-k q4_0 \
    --threads 16 \
    --prio 2 \
    --temp 0.6 \
    --ctx-size 6000 \
    --n-gpu-layers 62 \
    --seed 3407 \
    -no-cnv \
    --prompt "<｜User｜>Create a Flappy Bird game in Python.<｜Assistant｜>"
```
If you get CUDA allocation error, please try to reduce `--n-gpu-layers` or `--ctx-size`.


---
### 4.2. Inference via Open WebUI
#### 4.2.1. Model Serving
To use ChatGPT style WebUI, you first need to serve your local DeepSeek model.
```bash 
# You can set your own model path at the '--model' option. This is just an example.
CUDA_VISIBLE_DEVICES=0,1 ./llama.cpp/llama-server \
    --model DeepSeek-R1-GGUF/DeepSeek-R1-UD-IQ1_S/DeepSeek-R1-UD-IQ1_S-00001-of-00003.gguf \
    --port 1234 \
    --ctx-size 1024 \
    --n-gpu-layers 62
```
If you get CUDA allocation error, please try to reduce `--n-gpu-layers` or `--ctx-size`.


#### 4.2.2. WebUI Serving
In a new terminal, connect to the same container as above and follow the commands below:
```bash
pip3 install open-webui
open-webui serve    # automatically serveing at http://localhost:8080
```

Then, please access the external port 7777 connected to 8080 via `http://{SERVER_URL}:7777`. After a simple account registration, navigate to `Profile > Settings > Admin Settings > Connections` in the top-right corner and add the following information.
* `http://127.0.0.1:1234`
* key: none

Then, you can chat with 671B DeepSeek!

<br><br>


## Acknowledgement
* [Unsloth](https://unsloth.ai/blog/deepseekr1-dynamic#running%20r1)
* [llama.cpp](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md)
* [Open WebUI](https://docs.openwebui.com/tutorials/integrations/deepseekr1-dynamic/#step-3-make-sure-open-webui-is-installed-and-running)