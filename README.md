# Idle

Idle is a local, voice-driven IoT assistant built on ESP32 boards. You talk to it, it understands you with speech recognition, thinks with a local language model, answers with a synthesized voice, reads your sensors, looks through a camera, and can even send an SMS or place a call through a phone. Everything runs on your own machines, with no paid cloud service.

## What it does

Idle listens to a spoken request, turns it into text, and decides what to do. Simple commands such as switching a light or reading the temperature are caught by a rule-based intent parser, so they answer fast. Open questions go to a local language model. The answer is spoken back, sentence by sentence, so the voice starts before the whole reply is ready. A camera board lets Idle detect people and objects and read QR codes, and a presence module notices when someone arrives. A bridge app on an Android phone lets the assistant send messages and make calls to the contacts you define.

## How it works

Two ESP32 boards talk to a Mosquitto MQTT broker over Wi-Fi. The first, called idle-node, runs on an ESP32-WROOM-32D and handles the sensors and actuators. The second, called idle-cam, is an ESP32-CAM that provides images. A FastAPI backend subscribes to the broker and runs the whole pipeline: faster-whisper for speech to text, Ollama running Llama 3.2 for the language model, Piper for text to speech, and Ultralytics YOLO with OpenCV for vision. The backend also exposes a WebSocket and a few HTTP endpoints for live communication. For phone features, the backend calls a small Python script running in Termux on the phone.

## Repository layout

```
idle-iot/
  firmware/idle-node/   ESP32-WROOM-32D firmware (PlatformIO)
  firmware/idle-cam/    ESP32-CAM firmware (PlatformIO)
  backend/              FastAPI app, voice and vision pipeline, tests
  infra/mosquitto/      MQTT broker configuration
  phone/                Termux bridge for SMS and calls
  docs/                 design decisions, benchmarks, images
  docker-compose.yml    starts the MQTT broker
```

## Getting started

You need Git, Docker, Python 3, Ollama, and VS Code with the PlatformIO extension. The commands below are written for Git Bash on Windows, and they work the same on Linux and macOS except for the virtual environment line.

First, clone the repository:

```
git clone git@github.com:Shawarma-Frikz/idle-iot.git
cd idle-iot
```

Second, create your private configuration files. They are ignored by Git, so your passwords never reach GitHub:

```
cp backend/.env.example backend/.env
cp backend/contacts.example.json backend/contacts.json
cp firmware/idle-node/include/config.example.h firmware/idle-node/include/config.h
cp firmware/idle-cam/include/config.example.h firmware/idle-cam/include/config.h
```

Open each copied file and fill in your own values, such as your Wi-Fi name and password, the IP address of the computer running the broker, and your contacts. Each example file explains its own fields.

Third, start the MQTT broker:

```
docker compose up -d
```

Fourth, install and start the backend:

```
cd backend
python -m venv .venv
source .venv/Scripts/activate
pip install -r requirements.txt
ollama pull llama3.2
uvicorn app.main:app --reload
```

On Linux or macOS, replace the `source` line with `source .venv/bin/activate`. The speech model downloads itself the first time it runs. The Piper voice file must be downloaded separately, and its location goes in `backend/.env`.

Fifth, flash the boards. In VS Code, open the `firmware/idle-node` folder, connect the board with a USB cable, and click the PlatformIO upload arrow in the bottom bar. Do the same for `firmware/idle-cam`.

Sixth, set up the phone bridge by following the notes in `phone/README.md`.

## Tests

The tests cover the intent parser and the sentence splitter. From the `backend` folder, with the virtual environment active, run:

```
pip install -r requirements-dev.txt
pytest
```

## Documentation

Each important design choice is recorded as a short file in `docs/decisions`. Measured latency and memory figures are collected in `docs/benchmarks.md`. Wiring photos and the architecture diagram are in `docs/images`.

## Status

Idle is an early-stage personal project under active development. Features and wiring may change between commits.

## Licenses and credits

Idle's own code is released under the MIT License. The LICENSE file at the root of the repository contains the full text, with the copyright line "Copyright (c) 2026 Idris".

Third-party components keep their own licenses. Ultralytics YOLO is AGPL-3.0, which is fine for a personal or academic project, but anyone who distributes or hosts a product built on it must follow the AGPL terms or swap in a permissively licensed detector. Whisper and faster-whisper are MIT. Piper and its voices have their own licenses: check the license of the Piper repository you install from and the license listed for each voice you download. Ollama and llama.cpp are MIT, and the Llama 3.2 model is covered by the Meta Llama 3.2 Community License. Eclipse Mosquitto is dual-licensed under EPL-2.0 and EDL-1.0. FastAPI is MIT. OpenCV is Apache-2.0. PlatformIO Core is Apache-2.0, the Arduino-ESP32 framework is LGPL-2.1, and the DHT sensor library by Adafruit is MIT. The Termux app used by the phone bridge is GPL-3.0.

Thanks to the open-source communities behind ESP32, PlatformIO, Whisper, Piper, Ollama, YOLO, Mosquitto, FastAPI, OpenCV and Termux.