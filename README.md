# Colab Drive benchmarks

How fast can a fresh Colab VM load model files from Google Drive? These notebooks test checkpoint persistence and time sequential reads from a mounted Drive to local VM disk. Most runs used the Colab extension for VS Code.

## Results

### Pika H3 Flex

The full checkpoint contains five files totaling 42.03 GB. The A100 read test used two of those files, so the matched comparison uses their timings from the full CPU read.

| Cold Drive-to-disk read | Data copied | Time | Throughput |
| --- | ---: | ---: | ---: |
| Colab CPU VM, full checkpoint | 42.03 GB | 19m 32s | 35.9 MB/s |
| Colab CPU VM, matched files | 4.77 GB | 2m 09s | 37.0 MB/s |
| Colab A100 80 GB VM, matched files | 4.77 GB | 1m 07s | 71.1 MB/s |

The matched files were the 1.96 GB Pika adapter and the 2.81 GB video VAE. The A100 VM read the pair about 1.9× as fast in these runs. At the measured A100 sample rate, a full 42.03 GB copy would take about 9m 51s. That figure is an extrapolation.

On a separate A100 run, downloading the full checkpoint from Hugging Face and copying it to Drive took **4m 41s** before the final Drive flush. After ComfyUI started, the first 768 × 448, 56-frame clip took **88.5 seconds** from workflow submission to completion. The two timing tests cover different transfer paths. The cold-read timer starts after Drive authorization and measures sequential copy time to the VM's local disk. These are individual Colab VM measurements, and throughput can vary by session. [Colab FAQ](https://research.google.com/colaboratory/faq.html)

For this public checkpoint, downloading from Hugging Face was the more promising path for a fresh session. Drive remains useful for archiving checkpoints, fine-tunes, and outputs. A CPU VM can download and archive weights before a GPU is allocated.

### SmolLM2

- **135M Instruct.** Saved the checkpoint from a Colab CPU VM in VS Code, then generated text from Drive on a fresh CPU VM in browser Colab.
- **360M Instruct.** Saved the checkpoint on a Colab Tesla T4 through VS Code. A second T4 generated the same reply from the Drive copy with Hugging Face offline.

## Run the notebooks

Install the [Colab extension](https://github.com/googlecolab/colab-vscode) in VS Code, open a notebook, and use **Select Kernel → Colab → New Colab Server**. Each notebook identifies its runtime type, Drive destination, and cell order. The SmolLM2 CPU notebook also includes instructions for browser Colab.

| Notebook | What to run |
| --- | --- |
| [SmolLM2 135M persistence](colab-model-persistence.ipynb) | An initial CPU run and a reload on a fresh VM. |
| [SmolLM2 360M GPU persistence](colab-gpu-model-persistence.ipynb) | A CUDA baseline and a Drive-only reload on a fresh T4. |
| [Pika H3 Flex setup and render](colab-pika-h3-flex-drive-benchmark.ipynb) | Save the pinned ComfyUI weights to Drive and render a baseline. The notebook also includes a full cold-reload cell for a fresh A100. |
| [Pika H3 Flex full CPU Drive read](colab-pika-h3-cold-drive-read.ipynb) | Read the saved 42.03 GB checkpoint on a fresh CPU VM. |
| [Pika H3 Flex A100 Drive sample](colab-pika-h3-a100-drive-read.ipynb) | Read the matching 4.77 GB subset on a fresh A100. |

The Pika read notebooks use the saved checkpoint and baseline record from the setup notebook. They mount Drive read-only and remove the temporary copies after timing. Remove the Colab server after each run when you are done with it.

The Pika H3 Flex weights are governed by the [MiniMax H3 Community License](https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/LICENSE). The Pika notebook uses the [ComfyUI Pika H3 Flex text-to-video workflow](https://docs.comfy.org/tutorials/video/pika/pika-h3-flex). The tests were inspired by [Kaan’s Colab and ComfyUI thread](https://x.com/kaanakz/status/2119098278365278486).
