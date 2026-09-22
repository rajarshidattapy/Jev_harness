🚀 I turned a Kaggle GPU into my own remote AI coding backend.
No OpenAI API.
No Anthropic API.
No expensive inference endpoint.
⚙️ Qwen3 8B + Ollama + Kaggle GPU + Cloudflare Tunnel + Cline
Architecture:
💻 Cline / VS Code
↓
☁️ Cloudflare Tunnel
↓
🖥️ Kaggle GPU
↓
⚡ Ollama
↓
🧠 Qwen3 8B

🔗 GitHub: https://lnkd.in/gGZekYF6

What makes this interesting? 👇
I’m not just running an LLM inside a notebook.
I connected my local Cline environment on Mac directly to a Qwen3 model running remotely on a Kaggle GPU through an OpenAI-compatible API.
✅ /v1/models → 200
✅ /v1/chat/completions → 200
✅ Remote inference → Working
✅ Cline → Connected
🔥 The interesting part: a free/temporary GPU environment can be turned into a remote inference layer that your local AI coding tools can actually use.
This opens up an interesting direction:
🔐 Authentication
🌐 Persistent hosting
📊 Monitoring
⚡ Scalable inference
Open-source models + accessible GPU infrastructure = a very interesting AI stack.

https://github.com/pruthviraj-chavan/kaggle_32gpu_free