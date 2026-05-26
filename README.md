# OpenWebUI와 Ollama를 활용한 Local LLM Stack 구축

## 1. Open Local LLM Stack 레이어 정의

| Layer | 역할 | 오픈소스 후보 | 이번 구축에서 선택 |
|---|---|---|---|
| User Interface Layer | 사용자가 브라우저에서 LLM과 대화하는 화면 제공 | OpenWebUI, LibreChat, AnythingLLM | OpenWebUI |
| API / Model Serving Layer | 로컬 모델을 API 형태로 제공하고 UI와 연결 | Ollama, TabbyAPI, vLLM, SGLang, llama.cpp server | Ollama |
| Inference Runtime Layer | 실제 모델 추론 실행 | Ollama Runtime, llama.cpp, ExLlamaV3, Transformers | Ollama Runtime |
| Model Format Layer | 모델 파일 형식 및 양자화 포맷 | GGUF, EXL3, GPTQ, AWQ | GGUF |
| Model Layer | 실제 사용할 LLM 모델 | qwen2.5:1.5b, llama3.2:1b, Qwen3, Mistral, Gemma | qwen2.5:1.5b |
| Model Repository Layer | 모델 다운로드 및 관리 | Ollama Library, Hugging Face Hub | Ollama Library |
| Runtime / Container Layer | 실행 환경 구성 및 재현성 확보 | Docker, WSL, Python venv, Conda | WSL + Docker |
| Hardware Acceleration Layer | 추론 연산 가속 | NVIDIA CUDA, AMD ROCm, CPU | NVIDIA GPU / CUDA |

## 2. 선택한 실행 환경

이번 실습에서는 Open Local LLM Stack을 구축하기 위해 UI 레이어에는 OpenWebUI를, API 및 Model Serving 레이어에는 Ollama를 선택하였다. 모델은 로컬 환경에 이미 설치되어 있던 qwen2.5:1.5b와 llama3.2:1b 중에서 qwen2.5:1.5b를 선택하였다.

OpenWebUI는 브라우저 기반 사용자 인터페이스를 제공하고, Ollama는 로컬 LLM 모델을 실행하며 API를 통해 OpenWebUI와 연결된다. 실제 모델 추론은 Ollama Runtime에서 수행되며, 실행 환경은 WSL과 Docker를 기반으로 구성하였다.

## 3. 전체 아키텍처

```text
User
 ↓
Browser
 ↓
OpenWebUI
 ↓
Ollama API
 ↓
Ollama Runtime
 ↓
qwen2.5:1.5b
 ↓
Local Hardware / NVIDIA GPU
## 4. 실행 과정
### 4.1 Ollama 모델 실행 확인
```text
WSL 환경에서 Ollama를 실행하고 qwen2.5:1.5b 모델을 구동하였다.

ollama run qwen2.5:1.5b

실행 결과 qwen2.5:1.5b 모델이 정상적으로 로드되었고, 프롬프트 입력 대기 상태가 표시되었다.
### 4.2 OpenWebUI Docker 실행
```text
OpenWebUI는 Docker를 사용하여 실행하였다.

docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
처음에는 Docker 명령어가 WSL에서 인식되지 않았으나, Docker Desktop의 WSL Integration을 활성화한 후 정상적으로 실행할 수 있었다.
### 4.3 OpenWebUI 접속 확인

```text
브라우저에서 다음 주소로 접속하였다.
http://127.0.0.1:3000
처음에는 http://localhost:3000 으로 접속했을 때 페이지가 표시되지 않았지만, 127.0.0.1 주소를 사용하여 접속 문제를 해결하였다.
### 4.4 OpenWebUI와 Ollama 연결 확인
```text
OpenWebUI 화면에서 qwen2.5:1.5b 모델이 표시되었고, 이를 통해 OpenWebUI가 Ollama API와 정상적으로 연결되었음을 확인하였다.

## 5. 실행 결과
OpenWebUI에서 qwen2.5:1.5b 모델을 선택할 수 있었고, 브라우저 기반 UI를 통해 로컬 LLM 모델을 사용할 수 있는 환경이 구축되었다.

## 6. 결론
```text
이번 실습에서는 OpenWebUI와 Ollama를 활용하여 로컬 LLM 실행 환경을 구축하였다. OpenWebUI는 사용자가 브라우저에서 모델과 대화할 수 있는 UI 역할을 수행하고, Ollama는 로컬 모델을 실행하며 API를 통해 OpenWebUI와 연결되는 Serving Layer 역할을 수행한다.

실습 모델로는 qwen2.5:1.5b를 선택하였다. 또한 Docker를 통해 OpenWebUI를 실행하고, WSL 환경에서 Ollama를 사용하여 로컬 모델이 정상적으로 동작하는 것을 확인하였다. 최종적으로 OpenWebUI 화면에서 qwen2.5:1.5b 모델이 표시되어 전체 Local LLM Stack이 정상적으로 구성되었음을 확인하였다.
```text
