1. Download the model & prepare the script. Use `hf` for downloading quickly but downgrade is required later.
```
~$ python3 -m venv envs/wan
~$ source envs/wan/bin/activate

~$ git clone https://github.com/Wan-Video/Wan2.2.git
~$ cd Wan2.2

~$ pip install --default-timeout=300 --retries 10 torch torchvision --index-url https://download.pytorch.org/whl/cu130
~$ pip install --default-timeout=300 --retries 10 -r requirements.txt

~$ pip install "huggingface_hub[cli]" hf_transfer
~$ export HF_HUB_ENABLE_HF_TRANSFER=1
~$ hf download Wan-AI/Wan2.2-TI2V-5B --local-dir ./Wan2.2-TI2V-5B --resume-download
```

2. Build `flash_attn` if required. Make sure the versions of `torch` & `nvcc` are matched. 
```
export CUDA_HOME=/usr/local/cuda-13.0
export PATH=/usr/local/cuda-13.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-13.0/lib64:${LD_LIBRARY_PATH}

hash -r

rm -rf build dist *.egg-info
rm -rf ~/.cache/pip ~/.cache/torch_extensions

python - <<'PY'
import os
import torch
from torch.utils.cpp_extension import CUDA_HOME
print("torch:", torch.__version__)
print("torch.version.cuda:", torch.version.cuda)
print("env CUDA_HOME:", os.environ.get("CUDA_HOME"))
print("torch CUDA_HOME:", CUDA_HOME)
PY

MAX_JOBS=16 CUDA_HOME=/usr/local/cuda-13.0 \
PATH=/usr/local/cuda-13.0/bin:$PATH \
LD_LIBRARY_PATH=/usr/local/cuda-13.0/lib64:$LD_LIBRARY_PATH \
pip install --no-build-isolation --no-cache-dir .
```

3. Run text-image-to-video generation.
```
~$ python -m pip uninstall -y huggingface_hub
~$ python -m pip install "huggingface_hub>=0.34,<1.0"
~$ pip install decord librosa peft

~$ python generate.py --task ti2v-5B --size 1280*704 --ckpt_dir ./Wan2.2-TI2V-5B --offload_model True --convert_model_dtype --t5_cpu --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage"
```
