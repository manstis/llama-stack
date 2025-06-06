---
orphan: true
---

# TGI Distribution

```{toctree}
:maxdepth: 2
:hidden:

self
```

The `llamastack/distribution-{{ name }}` distribution consists of the following provider configurations.

{{ providers_table }}

You can use this distribution if you have GPUs and want to run an independent TGI server container for running inference.

{% if run_config_env_vars %}
### Environment Variables

The following environment variables can be configured:

{% for var, (default_value, description) in run_config_env_vars.items() %}
- `{{ var }}`: {{ description }} (default: `{{ default_value }}`)
{% endfor %}
{% endif %}


## Setting up TGI server

Please check the [TGI Getting Started Guide](https://github.com/huggingface/text-generation-inference?tab=readme-ov-file#get-started) to get a TGI endpoint. Here is a sample script to start a TGI server locally via Docker:

```bash
export INFERENCE_PORT=8080
export INFERENCE_MODEL=meta-llama/Llama-3.2-3B-Instruct
export CUDA_VISIBLE_DEVICES=0

docker run --rm -it \
  --pull always \
  -v $HOME/.cache/huggingface:/data \
  -p $INFERENCE_PORT:$INFERENCE_PORT \
  --gpus $CUDA_VISIBLE_DEVICES \
  ghcr.io/huggingface/text-generation-inference:2.3.1 \
  --dtype bfloat16 \
  --usage-stats off \
  --sharded false \
  --cuda-memory-fraction 0.7 \
  --model-id $INFERENCE_MODEL \
  --port $INFERENCE_PORT
```

If you are using Llama Stack Safety / Shield APIs, then you will need to also run another instance of a TGI with a corresponding safety model like `meta-llama/Llama-Guard-3-1B` using a script like:

```bash
export SAFETY_PORT=8081
export SAFETY_MODEL=meta-llama/Llama-Guard-3-1B
export CUDA_VISIBLE_DEVICES=1

docker run --rm -it \
  --pull always \
  -v $HOME/.cache/huggingface:/data \
  -p $SAFETY_PORT:$SAFETY_PORT \
  --gpus $CUDA_VISIBLE_DEVICES \
  ghcr.io/huggingface/text-generation-inference:2.3.1 \
  --dtype bfloat16 \
  --usage-stats off \
  --sharded false \
  --model-id $SAFETY_MODEL \
  --port $SAFETY_PORT
```

## Running Llama Stack

Now you are ready to run Llama Stack with TGI as the inference provider. You can do this via Conda (build code) or Docker which has a pre-built image.

### Via Docker

This method allows you to get started quickly without having to build the distribution code.

```bash
LLAMA_STACK_PORT=8321
docker run \
  -it \
  --pull always \
  -p $LLAMA_STACK_PORT:$LLAMA_STACK_PORT \
  llamastack/distribution-{{ name }} \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env TGI_URL=http://host.docker.internal:$INFERENCE_PORT
```

If you are using Llama Stack Safety / Shield APIs, use:

```bash
# You need a local checkout of llama-stack to run this, get it using
# git clone https://github.com/meta-llama/llama-stack.git
cd /path/to/llama-stack

docker run \
  -it \
  --pull always \
  -p $LLAMA_STACK_PORT:$LLAMA_STACK_PORT \
  -v ~/.llama:/root/.llama \
  -v ./llama_stack/templates/tgi/run-with-safety.yaml:/root/my-run.yaml \
  llamastack/distribution-{{ name }} \
  --config /root/my-run.yaml \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env TGI_URL=http://host.docker.internal:$INFERENCE_PORT \
  --env SAFETY_MODEL=$SAFETY_MODEL \
  --env TGI_SAFETY_URL=http://host.docker.internal:$SAFETY_PORT
```

### Via Conda

Make sure you have done `uv pip install llama-stack` and have the Llama Stack CLI available.

```bash
llama stack build --template {{ name }} --image-type conda
llama stack run ./run.yaml
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env TGI_URL=http://127.0.0.1:$INFERENCE_PORT
```

If you are using Llama Stack Safety / Shield APIs, use:

```bash
llama stack run ./run-with-safety.yaml \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env TGI_URL=http://127.0.0.1:$INFERENCE_PORT \
  --env SAFETY_MODEL=$SAFETY_MODEL \
  --env TGI_SAFETY_URL=http://127.0.0.1:$SAFETY_PORT
```
