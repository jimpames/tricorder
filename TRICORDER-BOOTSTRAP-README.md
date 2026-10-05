# Tricorder bootstrap — donor tar and a blank Fold

How tube 2 was stood up from tube 1 on 4 October 2026. Same chip (Snapdragon 888, Galaxy Fold 3), same Termux ABI (`aarch64`). The tar is weights, the launcher, and the binaries. It is not Termux, and it is not the APK.

Reference donor home: `/data/data/com.termux/files/home`  
Finished archive: `/sdcard/Download/tricorder-backend.tgz` (5.6 GB, 5621427824 bytes)

## What the tar is

Included:

- `bin/tricorder` — launcher. It sets `LD_LIBRARY_PATH` to `~/llama.cpp/build/bin` before starting chat and vision.
- `models/` — Gemma 3 4B Q4, SmolVLM2 500M Q8, mmproj, SD 2.1 turbo Q4 (about 5.0 GB)
- `kokoro/` — ONNX weights, voices, and `server.py`
- `llama.cpp/build/bin/llama-server` — 7 KB ELF stub, not the server
- every `libllama*.so*`, `libggml*.so*`, and `libmtmd*.so*` next to that stub. This includes `libllama-common.so.0` (57 MB). The first tar missed it, and chat/vision died with `CANNOT LINK EXECUTABLE`.
- `sd-server` and `sd-cli`
- `prefix/lib/` — `libprotobuf.so`, `libprotobuf-lite.so`, `libre2.so*`, `libutf8_validity.so*`. ONNX Runtime's Android build is linked against these. They are not in the home tree.
- `prefix/lib/python3.14/site-packages/` — `onnxruntime`, `kokoro_onnx`, `espeakng_loader`, `phonemizer`, `dlinfo`, `cffi`, `_cffi_backend`, `soundfile`, `joblib`, `attrs`, `typing_extensions`. None of these have a Termux wheel. Do not `pip install numpy` or `pip install cffi` on the new phone.

Not included, on purpose:

- the rest of `$PREFIX`. The app UID differs on the new phone. Python, espeak, and NumPy are `pkg install`s.
- `storage`, `downloads`, `tricorder-logs`, `tricorder-pids`
- the Tricorder APK and the field log. The log moves with Comlink backup/restore.
- every `test-*` tool in the llama build tree
- `libc.so`, `libdl.so`, `liblog.so`. Those are Android system libraries. Copying them breaks the linker.

## 1. Tar the donor

SSH to the donor is port **8022**, not 22. Username is `whoami`. `passwd` sets the password. There is no second user.

Paste this on the donor. It is the master pack. It refuses to write the archive if `libllama-common.so.0`, `kokoro/server.py`, or `onnxruntime` is missing.

```bash
cd ~ && rm -rf tricorder-pack-stage && mkdir -p tricorder-pack-stage/llama.cpp/build/bin tricorder-pack-stage/stable-diffusion.cpp/build/bin tricorder-pack-stage/bin tricorder-pack-stage/prefix/lib/python3.14/site-packages && cp -a bin/tricorder tricorder-pack-stage/bin/ && cp -a models kokoro tricorder-pack-stage/ && cp -a llama.cpp/build/bin/llama-server llama.cpp/build/bin/libllama*.so* llama.cpp/build/bin/libggml*.so* llama.cpp/build/bin/libmtmd*.so* tricorder-pack-stage/llama.cpp/build/bin/ && cp -a stable-diffusion.cpp/build/bin/sd-server stable-diffusion.cpp/build/bin/sd-cli tricorder-pack-stage/stable-diffusion.cpp/build/bin/ && cp -a $PREFIX/lib/libprotobuf*.so* $PREFIX/lib/libre2.so* $PREFIX/lib/libutf8_validity.so* tricorder-pack-stage/prefix/lib/ && PY=$PREFIX/lib/python3.14/site-packages && for m in onnxruntime kokoro_onnx espeakng_loader phonemizer dlinfo cffi soundfile joblib attrs typing_extensions; do cp -a "$PY/$m" tricorder-pack-stage/prefix/lib/python3.14/site-packages/; done && cp -a "$PY"/_cffi_backend*.so "$PY"/soundfile.py "$PY"/_soundfile*.so tricorder-pack-stage/prefix/lib/python3.14/site-packages/ && test -f tricorder-pack-stage/llama.cpp/build/bin/libllama-common.so.0 && test -f tricorder-pack-stage/kokoro/server.py && test -d tricorder-pack-stage/prefix/lib/python3.14/site-packages/onnxruntime && tar -czf /sdcard/Download/tricorder-backend.tgz -C tricorder-pack-stage . && rm -rf tricorder-pack-stage && ls -lh /sdcard/Download/tricorder-backend.tgz
```

On the new phone, after Termux, storage permission, and the package install below:

```bash
pkg install -y python python-pip python-numpy espeak libsndfile clang cmake git wget libandroid-execinfo && cd ~ && rm -rf tricorder-unpack-stage && mkdir -p tricorder-unpack-stage && tar -xzf /sdcard/Download/tricorder-backend.tgz -C tricorder-unpack-stage && mkdir -p llama.cpp/build/bin stable-diffusion.cpp/build/bin bin && cp -a tricorder-unpack-stage/llama.cpp/build/bin/. llama.cpp/build/bin/ && cp -a tricorder-unpack-stage/stable-diffusion.cpp/build/bin/. stable-diffusion.cpp/build/bin/ && cp -a tricorder-unpack-stage/bin/tricorder bin/tricorder && cp -a tricorder-unpack-stage/models tricorder-unpack-stage/kokoro . && cp -a tricorder-unpack-stage/prefix/lib/. $PREFIX/lib/ && chmod +x bin/tricorder llama.cpp/build/bin/llama-server stable-diffusion.cpp/build/bin/sd-server stable-diffusion.cpp/build/bin/sd-cli && grep -q 'HOME/bin' ~/.bashrc || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc && grep -q PHONEMIZER_ESPEAK_LIBRARY ~/.bashrc || printf 'export PHONEMIZER_ESPEAK_LIBRARY=$PREFIX/lib/libespeak-ng.so\nexport ESPEAK_DATA_PATH=$PREFIX/share/espeak-ng-data\n' >> ~/.bashrc && export PATH="$HOME/bin:$PATH" && rm -rf tricorder-unpack-stage && tricorder start && sleep 15 && tricorder status
```

`python-numpy` comes from Termux, not pip. Pip compiling NumPy or cffi is what burned the first clone.

`llama-server` at about 7 KB is expected. Confirm it is ELF and that `libllama-server-impl.so` is tens of megabytes. Do not `head` the stub; it is binary and will dump garbage into the terminal. `reset` if that already happened.

Copy the tgz off the phone (USB, `scp -P 8022`, or a read-only Drive share). Leave a copy in Download on the donor.

## 2. New phone — Termux, not Play

Install **Termux from F-Droid**, not the abandoned Play listing. Install **Termux:API from F-Droid** as well if you want `termux-wake-lock`.

Android settings, both apps:

- Battery: Unrestricted
- Files and media: Allow (or run `termux-setup-storage` and accept the prompt)

Open Termux once so it finishes bootstrap. Then:

```bash
pkg install -y openssh
passwd
sshd
whoami
```

PuTTY: host is the phone Wi-Fi address, port **8022**, username is whatever `whoami` printed. Any name you type that is not that user is rejected. Same lab password from `passwd` is fine.

Keep the session awake:

```bash
pkg install -y termux-api && termux-wake-lock
```

`termux-wake-unlock` releases it. The Termux notification has the same toggle.

## 3. Mirror, then packages

A China mirror from the US East Coast can sit at 10 kB/s. Point apt at the official repo before installing:

```bash
printf 'deb https://packages.termux.dev/apt/termux-main stable main\n' > $PREFIX/etc/apt/sources.list && pkg update && pkg install -y python python-pip espeak clang cmake git wget libandroid-execinfo
```

If that is also slow, `termux-change-repo` and pick a mirror that is not in China. Do not untar until this finishes. The tar does not contain Python or espeak.

## 4. Storage permission, then untar

`/sdcard` is denied until storage is granted:

```bash
termux-setup-storage
```

Accept the prompt. If it does not appear: Settings, Apps, Termux, Permissions, Files and media.

```bash
ls -lh /sdcard/Download/tricorder-backend.tgz && cd ~ && tar -xzf /sdcard/Download/tricorder-backend.tgz && chmod +x ~/bin/tricorder ~/llama.cpp/build/bin/llama-server ~/stable-diffusion.cpp/build/bin/sd-server ~/stable-diffusion.cpp/build/bin/sd-cli && ls -lh ~/llama.cpp/build/bin/libllama-server-impl.so ~/models ~/kokoro/*.onnx
```

## 5. PATH

Termux does not put `~/bin` on `PATH`. The command is not missing; the shell cannot see it.

```bash
grep -q 'HOME/bin' ~/.bashrc || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
export PATH="$HOME/bin:$PATH"
hash -r
which tricorder
```

`which` should print `/data/data/com.termux/files/home/bin/tricorder`. A new SSH session picks this up from `.bashrc`. This session needs the `export` line.

## 6. Kokoro Python

Weights are in the tar. The interpreter packages are not.

```bash
cd ~/kokoro && pip install kokoro-onnx soundfile numpy
export PHONEMIZER_ESPEAK_LIBRARY=$PREFIX/lib/libespeak-ng.so
export ESPEAK_DATA_PATH=$PREFIX/share/espeak-ng-data
grep -q PHONEMIZER_ESPEAK_LIBRARY ~/.bashrc || cat >> ~/.bashrc << 'EOF'
export PHONEMIZER_ESPEAK_LIBRARY=$PREFIX/lib/libespeak-ng.so
export ESPEAK_DATA_PATH=$PREFIX/share/espeak-ng-data
EOF
```

Use `kokoro-v1.0.onnx` (fp32). The fp16 graph NaNs on this ONNX Runtime / Android pair.

## 7. Start

First start from a Termux session that already holds the wakelock. `tricorder start` also tries `termux-wake-lock` and starts sshd if it is down.

```bash
tricorder start
sleep 15
tricorder status
```

Expected: chat :8080, vision :8081, image :8188, tts :5000 up. Logs are `~/tricorder-logs/*.log`. `tricorder stop` does not kill sshd.

`CANNOT LINK EXECUTABLE` means the new Termux is too far from the donor. Rebuild only the binary there; do not re-fetch the weights. For sd-server, cmake with `-DSD_WEBP=OFF` avoids the missing `cpu-features` hole.

## 8. APK

Sideload separately. 0.6.94 is the build with the SUBSPACE rate slider and category boxes. The tar will not install it.

Android settings for the Tricorder app: camera, microphone, location, nearby devices, notifications, battery Unrestricted. Files access if backup/restore should see Download.

Comlink name, field log, and flight book do not come across in the tar. Use Comlink backup on the donor and restore on the new tube if you want the same book.

## 9. SUBSPACE cards

Region and the private TRICORDER channel are set once in the Meshtastic app on each T1000-E. This APK does not write the channel table. LongFast is rendezvous only. Alerts leave the phone at the Comlink slider rate (1–5 per minute), not at detect rate.
