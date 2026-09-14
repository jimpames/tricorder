# Project Tricorder — Termux backend install

This is the only document you need to stand up the four on-device
language / image / speech servers the APK talks to.

The APK is **not** the LLMs. The APK is the instrument face, YAMNet,
BirdNET, flora, translator, horizon, and logs. Chat, vision, Imagine,
and Kokoro live in **Termux** on loopback.

Do the first start **from the Termux app on the phone**, not from an SSH
session. SSH dies; the servers should not.

---

## What you end up with

| Role    | Binary / process                         | Model | Port |
|---------|------------------------------------------|-------|------|
| Chat    | `~/llama.cpp/build/bin/llama-server`     | Gemma 3 4B Instruct Q4_K_M | **8080** |
| Vision  | same `llama-server`, second process      | SmolVLM2 500M Q8 + mmproj f16 | **8081** |
| Imagine | `~/stable-diffusion.cpp/build/bin/sd-server` | SD 2.1 Turbo Q4_0 | **8188** |
| Voice   | `python3 ~/kokoro/server.py`             | Kokoro v1.0 ONNX + voices.bin | **5000** |
| SSH     | Termux `sshd`                            | — | **8022** |

All four LLM ports stay up together. Do not stop 8188 to “free RAM for
chat.” The APK expects all four.

Launcher: `~/bin/tricorder start|stop|restart|status|logs`

---

## 0. Phone prerequisites

- arm64 Android 8+ (Snapdragon 888 / Fold 3 class is the reference)
- 8 GB RAM is tight; 12 GB is comfortable
- Storage: ~8 GB free for weights + builds
- Wi-Fi for the first download
- **Termux from F-Droid**, never Play Store
  https://f-droid.org/packages/com.termux/
- Termux:API from F-Droid if you want `termux-wake-lock` extras
- Battery → Termux → **Unrestricted**
- Sideload the Tricorder APK after the backends answer `status`

---

## 1. Termux base

Open Termux on the phone.

```bash
termux-setup-storage          # accept the Android prompt
termux-change-repo            # pick a fast mirror if pkg is slow
pkg update && pkg upgrade -y
pkg install -y wget curl git clang cmake python \
  openssh termux-api espeak
termux-wake-lock
mkdir -p "$HOME/bin" "$HOME/models" "$HOME/kokoro" \
         "$HOME/tricorder-logs" "$HOME/tricorder-pids"
```

`espeak` is the Termux package name. It ships `libespeak-ng.so`.

---

## 2. Put the launcher on the phone

Copy `termux/tricorder` from the project zip onto the device.
Easiest path: download the source zip, unzip in shared storage, then:

```bash
cp /sdcard/Download/project-tricorder/project-tricorder/termux/tricorder \
   "$HOME/bin/tricorder"
chmod +x "$HOME/bin/tricorder"

# optional: keep fetch-models next to it
cp /sdcard/Download/project-tricorder/project-tricorder/termux/fetch-models.sh \
   "$HOME/bin/fetch-models.sh"
chmod +x "$HOME/bin/fetch-models.sh"
```

If `~/bin` is not on `PATH`:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> "$HOME/.bashrc"
source "$HOME/.bashrc"
```

The launcher already knows these paths:

```
~/llama.cpp/build/bin/llama-server
~/stable-diffusion.cpp/build/bin/sd-server
~/models/gemma-3-4b-it-Q4_K_M.gguf
~/models/SmolVLM2-500M-Video-Instruct-Q8_0.gguf
~/models/mmproj-SmolVLM2-500M-Video-Instruct-f16.gguf
~/models/stable-diffusion-v2-1-turbo-Q4_0.gguf
~/kokoro/server.py
```

Do not rename the GGUFs unless you edit `~/bin/tricorder` to match.

sshd is port **8022**. `tricorder stop` does **not** kill sshd.

---

## 3. Download the four weights

From Termux, with Wi-Fi up:

```bash
bash "$HOME/bin/fetch-models.sh"
```

Or from the unzipped project:

```bash
bash /sdcard/Download/project-tricorder/project-tricorder/termux/fetch-models.sh
```

About **6 GB**. `wget -c` resumes if Wi-Fi drops. Safe to run again.

### Gemma 401 / “invalid username or password”

Gemma is gated. In a browser, log into Hugging Face, open

https://huggingface.co/ggml-org/gemma-3-4b-it-GGUF

accept the Gemma license, then rerun `fetch-models.sh`.

If it still 401s, create a read token at huggingface.co/settings/tokens
and:

```bash
export HF_TOKEN=hf_xxxxxxxx
wget -c --header="Authorization: Bearer $HF_TOKEN" \
  -O "$HOME/models/gemma-3-4b-it-Q4_K_M.gguf.part" \
  "https://huggingface.co/ggml-org/gemma-3-4b-it-GGUF/resolve/main/gemma-3-4b-it-Q4_K_M.gguf"
mv "$HOME/models/gemma-3-4b-it-Q4_K_M.gguf.part" \
   "$HOME/models/gemma-3-4b-it-Q4_K_M.gguf"
```

Do **not** use `llama-server -hf ...` as the daily driver. It is a
one-shot fetch. The launcher wants a file on disk at the path above.

### Kokoro weights — use fp32

`fetch-models.sh` pulls:

- `kokoro-v1.0.onnx` (fp32, PocketPal-class graph)
- `voices-v1.0.bin`

**Do not substitute `kokoro-v1.0.fp16.onnx`.** On this Android /
onnxruntime combo the fp16 graph returns NaN audio (silent 94 KB wav).

---

## 4. Binaries — copy first, compile only if you must

### 4a. Copy from a working phone (preferred)

On the donor Fold:

```bash
mkdir -p ~/export-bin
cp ~/llama.cpp/build/bin/llama-server ~/export-bin/
cp ~/stable-diffusion.cpp/build/bin/sd-server ~/export-bin/
tar -czf /sdcard/Download/tricorder-bins-aarch64.tgz -C ~ export-bin
```

On the new phone, after §1:

```bash
tar -xzf /sdcard/Download/tricorder-bins-aarch64.tgz -C ~
mkdir -p ~/llama.cpp/build/bin ~/stable-diffusion.cpp/build/bin
cp ~/export-bin/llama-server ~/llama.cpp/build/bin/
cp ~/export-bin/sd-server    ~/stable-diffusion.cpp/build/bin/
chmod +x ~/llama.cpp/build/bin/llama-server \
         ~/stable-diffusion.cpp/build/bin/sd-server
```

These ELFs are Termux aarch64 only. They will not run on stock Android,
iOS, or x86 Termux.

If you see `CANNOT LINK EXECUTABLE`, Termux libc does not match. Build
on that phone (§4b).

### 4b. Build on this phone

```bash
cd ~
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build -j --target llama-server

cd ~
git clone https://github.com/leejet/stable-diffusion.cpp
cd stable-diffusion.cpp
cmake -B build -DSD_WEBP=OFF
cmake --build build -j --target sd-server
```

`-DSD_WEBP=OFF` avoids the missing `cpu-features` / `cpufeatures-webp`
hole on Termux NDK.

CPU only. Adreno / OpenCL offload is a later rebuild, not required.

---

## 5. Kokoro Python (port 5000)

Kokoro is the painful one. Use this recipe; skip random pip pins.

```bash
pkg install -y espeak
export PHONEMIZER_ESPEAK_LIBRARY="$PREFIX/lib/libespeak-ng.so"
export ESPEAK_DATA_PATH="$PREFIX/share/espeak-ng-data"
echo 'export PHONEMIZER_ESPEAK_LIBRARY="$PREFIX/lib/libespeak-ng.so"' >> "$HOME/.bashrc"
echo 'export ESPEAK_DATA_PATH="$PREFIX/share/espeak-ng-data"' >> "$HOME/.bashrc"

pip install kokoro-onnx soundfile numpy
```

Termux Python 3.12–3.14 works. Do **not** downgrade Python to chase
`kokoro-onnx==0.6.1`. That wheel wants `espeakng-loader` which does not
install here.

If `import phonemizer` dies on `undefined symbol: dlinfo`, stub it:

```bash
python3 - <<'PY'
from pathlib import Path
p = Path("/data/data/com.termux/files/usr/lib/python3.14/site-packages/dlinfo")
p.mkdir(parents=True, exist_ok=True)
(p / "_glibc.py").write_text(
"""class DLInfo:
    def __init__(self, *args, **kwargs):
        import os
        self.path = os.environ.get(
            "PHONEMIZER_ESPEAK_LIBRARY",
            "/data/data/com.termux/files/usr/lib/libespeak-ng.so",
        )
"""
)
(p / "__init__.py").write_text("from dlinfo._glibc import DLInfo\n")
print("stubbed", p)
PY
```

Adjust `python3.14` in that path if `python3 -V` is different.

### server.py

If you already have a working `~/kokoro/server.py` from the Fold, copy
the whole `~/kokoro` directory (script + onnx + voices). Otherwise put
this file at `~/kokoro/server.py`:

```python
#!/usr/bin/env python3
"""Minimal Kokoro HTTP server for Project Tricorder. Port 5000."""
import io, json, os, traceback
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer
import numpy as np
import soundfile as sf

os.environ.setdefault(
    "PHONEMIZER_ESPEAK_LIBRARY",
    os.environ.get("PREFIX", "/data/data/com.termux/files/usr") + "/lib/libespeak-ng.so",
)
os.environ.setdefault(
    "ESPEAK_DATA_PATH",
    os.environ.get("PREFIX", "/data/data/com.termux/files/usr") + "/share/espeak-ng-data",
)

ROOT = os.path.dirname(os.path.abspath(__file__))
ONNX = os.path.join(ROOT, "kokoro-v1.0.onnx")
VOICES = os.path.join(ROOT, "voices-v1.0.bin")
HOST, PORT = "127.0.0.1", 5000

from kokoro_onnx import Kokoro
K = Kokoro(ONNX, VOICES)

def synth(text, voice="af_bella", speed=1.0):
    phonemes = K.tokenizer.phonemize(text, lang="en-us")
    tokens = K.tokenizer.tokenize(phonemes)
    pack = np.asarray(K.voices[voice])
    idx = min(max(len(tokens) - 1, 0), pack.shape[0] - 1)
    style = pack[idx]          # (1, 256) — do not pass the full (510,1,256)
    audio, _ = K._infer(tokens, style, float(speed))
    arr = np.asarray(audio).astype(np.float32).ravel()
    arr = np.nan_to_num(arr, nan=0.0, posinf=0.0, neginf=0.0)
    peak = float(np.max(np.abs(arr))) if arr.size else 0.0
    if peak > 1.0:
        arr = arr / peak
    return arr, 24000

class H(BaseHTTPRequestHandler):
    def log_message(self, *a):
        print(*a)
    def _send(self, code, body, ctype="application/json"):
        self.send_response(code)
        self.send_header("Content-Type", ctype)
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)
    def do_GET(self):
        if self.path.startswith("/health"):
            self._send(200, json.dumps({"status": "ok", "model": "kokoro-82m"}).encode())
        else:
            self._send(404, b'{"error":"not found"}')
    def do_POST(self):
        n = int(self.headers.get("Content-Length") or 0)
        raw = self.rfile.read(n) if n else b"{}"
        try:
            body = json.loads(raw.decode() or "{}")
        except Exception:
            body = {}
        text = (body.get("text") or body.get("input") or "").strip()
        if not text:
            self._send(400, b'{"error":"empty text"}'); return
        try:
            samples, sr = synth(text, body.get("voice") or "af_bella", body.get("speed") or 1.0)
            buf = io.BytesIO()
            sf.write(buf, samples, sr, format="WAV")
            self._send(200, buf.getvalue(), "audio/wav")
        except Exception as e:
            traceback.print_exc()
            self._send(500, json.dumps({"error": str(e)}).encode())

if __name__ == "__main__":
    print("kokoro on http://%s:%s" % (HOST, PORT), flush=True)
    ThreadingHTTPServer((HOST, PORT), H).serve_forever()
```

Dry run before the launcher:

```bash
cd ~/kokoro
python3 - <<'PY'
from kokoro_onnx import Kokoro
import numpy as np, soundfile as sf
k = Kokoro("kokoro-v1.0.onnx", "voices-v1.0.bin")
phonemes = k.tokenizer.phonemize("Tricorder online.", lang="en-us")
tokens = k.tokenizer.tokenize(phonemes)
pack = np.asarray(k.voices["af_bella"])
style = pack[min(len(tokens)-1, pack.shape[0]-1)]
audio, _ = k._infer(tokens, style, 1.0)
arr = np.nan_to_num(np.asarray(audio).astype("float32").ravel())
sf.write("/sdcard/Download/kokoro-test.wav", arr, 24000)
print("wrote wav", arr.size, float(np.max(np.abs(arr))))
PY
```

Play that wav from Files. You want a voice, not silence.

Style rank matters: pass `(1, 256)`, not the full `(510, 1, 256)` pack.
Wrong rank → empty or NaN audio.

---

## 6. First start

From the **Termux app**:

```bash
termux-wake-lock
~/bin/tricorder start
# wait 10–20 seconds
~/bin/tricorder status
```

Healthy example:

```
chat      :8080  up pid …          {"status":"ok"}
vision    :8081  up pid …          {"status":"ok"}
image     :8188  up pid …          Stable Diffusion Server is running
tts       :5000  up pid …          {"status":"ok","model":"kokoro-82m"}
```

Logs:

```bash
~/bin/tricorder logs chat
~/bin/tricorder logs vision
~/bin/tricorder logs image
~/bin/tricorder logs tts
```

Then open the Tricorder APK. CHAT / VISION / IMAGE / VOICE lamps should
go green.

SSH from the desk (optional):

```
ssh -p 8022 <user>@<phone-ip>
```

Password is whatever you set with `passwd` in Termux. Do not use SSH as
the process that launched the four servers.

---

## 7. Everyday commands

```bash
~/bin/tricorder start      # idempotent; skips anything already up
~/bin/tricorder status
~/bin/tricorder restart
~/bin/tricorder stop       # leaves sshd running
~/bin/tricorder logs tts
```

After a phone reboot: open Termux, `termux-wake-lock`, `tricorder start`,
wait, then open the APK.

---

## 8. What the APK fetches by itself

On first Earth-link boot the APK can pull BirdNET, flora/fauna TFLite,
translator language packs, TLEs, and SOL weather. It **cannot** install
Termux, compile llama.cpp, or wget the four GGUF/ONNX files. That is
this document.

---

## 9. Troubleshooting

| Symptom | Fix |
|---|---|
| `pkg` / F-Droid confusion | Uninstall Play Store Termux. Install F-Droid Termux. |
| Gemma wget 401 | Accept Gemma license on HF, or `HF_TOKEN`. |
| `CANNOT LINK EXECUTABLE` | Rebuild llama-server / sd-server on that phone. |
| cmake `cpu-features` / `cpufeatures-webp` | `cmake -B build -DSD_WEBP=OFF` |
| `spawn.h` while building random TTS | Stop. Use the Python kokoro path above. |
| Kokoro silent wav / `absmax nan` | You are on fp16 or passed the full style pack. Use fp32 + `pack[idx]`. |
| `undefined symbol: dlinfo` | Install the stub in §5. |
| `piper_phonemize` missing | Ignore it. Use phonemizer + espeak + the stub. |
| APK lamps red, `status` green | Confirm ports 8080/8081/8188/5000. Do not bind `0.0.0.0` if you do not need it. |
| Servers die after you leave SSH | Start them from the Termux app + `termux-wake-lock`. Battery unrestricted. |
| Imagine doubled / muddy | App sliders: 512 and 4–6 steps. Server `--cfg-scale 1` is already in the launcher. |
| Two phones | Copy `export-bin` + `~/models` + `~/kokoro` + `~/bin/tricorder`. Same ABI only. |

---

## 10. License notes

- Gemma: Google Gemma license. Accept on Hugging Face before download.
- SmolVLM2: Apache-2.0 class weights from ggml-org.
- SD Turbo GGUF: check the Hugging Face repo card.
- Kokoro ONNX + voices: thewh1teagle / hexgrad Kokoro terms.

Project Tricorder is not affiliated with The Wand Company, Paramount
SkyDance, Desilu, CBS, Roddenberry Entertainment, or Wah Chang.
