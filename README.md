# Assistente de Voz Local com IA 🎙️🤖

Um assistente de voz totalmente offline, executado no seu próprio computador, capaz de escutar através do microfone, transcrever o áudio, gerar respostas inteligentes utilizando um grande modelo de linguagem (LLM) e sintetizar a resposta com uma voz natural em português brasileiro. Tudo isso **sem depender de chamadas a APIs na nuvem** (como OpenAI ou Google), garantindo total privacidade e funcionamento sem internet.

---

## 🛠️ Tecnologias Utilizadas

Este projeto integra diversas tecnologias de Inteligência Artificial de ponta rodando localmente:

### 1. **OpenAI Whisper (Speech-to-Text - STT)**
*   **O que é:** Um modelo avançado de reconhecimento de fala da OpenAI.
*   **Como é usado:** Utilizamos a biblioteca `openai-whisper` em Python para carregar o modelo localmente (versão `base`). Ele recebe o arquivo de áudio capturado do microfone e o converte para texto com alta precisão, lidando bem com sotaques e ruídos de fundo.

### 2. **Ollama + Modelo Phi/Llama (Large Language Model - LLM)**
*   **O que é:** O Ollama é uma ferramenta leve construída em Go e C++ para rodar LLMs abertos localmente no seu computador.
*   **Como é usado:** O script `main.py` faz requisições HTTP (`requests`) para a API local do Ollama (`http://localhost:11434/api/generate`). O modelo padrão configurado é o **Microsoft Phi** (`phi:latest` - 1.6 GB) devido ao seu excelente balanço entre velocidade de inferência e capacidade intelectual em hardwares domésticos. Ele interpreta a transcrição do Whisper e gera a resposta contextualizada do assistente.

### 3. **Piper TTS (Text-to-Speech - TTS)**
*   **O que é:** Um motor neural incrivelmente rápido e leve de síntese de voz (desenvolvido pelo projeto Rhasspy/Home Assistant).
*   **Como é usado:** Nós baixamos o binário do Piper para Windows e o executamos via linha de comando no Python submetendo o texto gerado pelo Ollama. Utilizamos um modelo acústico treinado especificamente para Português do Brasil (`pt_BR-faber-medium.onnx`), o que produz uma voz masculina natural e limpa.

### 4. **Python (Backend & Orquestração)**
*   `sounddevice` & `scipy`: Capturam o áudio em tempo real diretamente da placa de som do seu PC, gravando-o em memória como um array do NumPy e o salvando no formato `.wav`.
*   `subprocess`: Usado para invocar dinamicamente o executável isolado do Piper passando corretamente as trilhas fonéticas (espeak-ng-data).

---

## 📂 Estrutura do Código

O projeto está organizado na seguinte estrutura principal:
```text
voice-ai/
│
├── audio/                      # Áudios temporários gravados e sintetizados (input.wav / output.wav)
├── models/
│   ├── piper/                  # Binários do motor de voz Piper e dados fonéticos (espeak)
│   └── voices/                 # Modelo de voz ONNX do Piper (pt_BR-faber-medium)
│
├── main.py                     # Script principal (O loop e cérebro do assistente)
├── piper_setup.py              # Script utilitário para baixar e instalar o Piper TTS automaticamente
├── requirements.txt            # Dependências PIP Python
└── README.md                   # Esta documentação
```

---

## ⚙️ Funcionamento Detalhado (O Loop Principal)

Toda a mágica acontece dentro da função `loop_principal()` no arquivo `main.py`. O fluxo é sequencial e contínuo até ser interrompido com `Ctrl+C`:

1.  **🎙️ Captura de Áudio (Ouvir):**
    O script usa a biblioteca `sounddevice` para gravar um áudio raw de até 6 segundos. Um pequeno filtro checa a potência do volume (RMS): se o áudio for muito baixo (abaixo de `0.01`), ele assume que houve **silêncio** e economiza recursos abortando a interação, voltando a ouvir prontamente.
    *O áudio captado é salvo em `audio/input.wav`.*

2.  **📝 Transcrição (Entender):**
    A função `transcrever()` entrega o `input.wav` ao modelo Whisper mantido em memória RAM ativa. O Whisper aplica inteligência artificial na forma de ondas e converte as palavras ditas para uma string de texto, focando no idioma `pt`.

3.  **🧠 Raciocínio (Pensar):**
    Com o texto do usuário transcrito, o script passa tudo como um prompt para o LLM. A função `perguntar_llm()` cria um *payload* injetando um **System Prompt** ("Você é um assistente técnico, direto...") mais a pergunta atual. Esta string é enviada à interface local do **Ollama**. O modelo calcula os tokens da resposta em background, retornando o texto da resposta.

4.  **🔊 Síntese de Voz (Falar):**
    Com a resposta pronta de texto em mãos, chamamos a função `sintetizar_voz()`. O script invoca ativamente o robô do **Piper** (`piper.exe`) repassando o texto. Em milésimos de segundo (geralmente sob `0.5s`), o Piper devolve um arquivo gerado `.wav` (`audio/output.wav`). Imediatamente o Python reproduz este arquivo recém-criado usando e bloqueia o loop via `sd.wait()` enquanto o assistente fala pelo alto-falante.

5.  Loop retoma para recomeçar o processo automaticamente ouvindo você.

---

## 🚀 Como Executar

Se tudo já estiver instalado de acordo com `piper_setup.py` e Ollama:

1. Abra o terminal.
2. Certifique-se que o assistente **Ollama** está ativo/rodando em background e o modelo está baixado (caso não, excute `ollama run phi` para forçar o download na primeira vez).
3. Execute:
   ```powershell
   python main.py
   ```
4. Assim que o console exibir `🎤 Ouvindo...`, fale sua pergunta no microfone.

Divirta-se criando conversas locais!
