# Estação Ambiental com ESP32

Sistema embarcado para **monitoramento de temperatura, umidade e concentração de CO₂**, desenvolvido como projeto acadêmico com ESP32-S3, display touch e sensores ambientais.

O projeto surgiu a partir de uma situação real de armazenamento de produtos químicos: alguns materiais precisam permanecer em condições controladas de temperatura, e uma falha no ar-condicionado pode fazer com que o ambiente fique fora da faixa esperada sem que isso seja percebido imediatamente. A proposta é transformar esse problema em uma solução de monitoramento contínuo e, em etapas futuras, evoluir para atuação e alertas automáticos.

> **Status atual:** a versão disponibilizada neste repositório implementa monitoramento local, interface gráfica, histórico e registro opcional em microSD. O acionamento automático do ar-condicionado, notificações remotas e o controle PID completo são evoluções planejadas e **não fazem parte da versão atual do firmware**.

---

## Objetivo

Desenvolver uma estação capaz de acompanhar continuamente as condições ambientais de uma sala e apresentar os dados de forma clara para o usuário.

### Variáveis monitoradas

- Temperatura em °C
- Umidade relativa em %
- Concentração de CO₂ em ppm

---

## Motivação

A ideia nasceu da necessidade de melhorar o controle das condições de armazenamento de produtos químicos utilizados em ambiente industrial.

No processo anterior, a temperatura era acompanhada por termômetro e os registros eram consultados posteriormente. Isso cria a possibilidade de uma falha permanecer despercebida por horas ou até dias.

A solução proposta busca criar uma base para:

1. visualizar as condições da sala em tempo real;
2. registrar o histórico das medições;
3. identificar situações fora da condição esperada;
4. futuramente acionar o sistema de climatização;
5. futuramente alertar os responsáveis;
6. futuramente utilizar estratégias de controle, incluindo PID, quando aplicável.

A imagem que registrou a origem da ideia está disponível em [`docs/Origem_da_Ideia.png`](docs/Origem_da_Ideia.png).

---

## Hardware utilizado

- ESP32-S3
- Placa/display **LCDWIKI ES3C28P**
- LCD ILI9341V, 240 × 320
- Touch capacitivo FT6336G
- Sensor de CO₂ **SCD41**
- Sensor de temperatura e umidade **SHT45**
- Cartão microSD opcional

### Barramento I²C

Os sensores SCD41 e SHT45 compartilham o mesmo barramento I²C utilizado pelo projeto:

| Dispositivo | Endereço |
|---|---:|
| SCD41 | `0x62` |
| SHT45 | `0x44` |
| FT6336G | `0x38` |

---

## Funcionalidades implementadas

- Leitura de CO₂, temperatura e umidade
- Interface gráfica touch com LVGL
- Tela principal com visão geral das variáveis
- Telas individuais para CO₂, temperatura e umidade
- Histórico gráfico de 1 h, 6 h e 24 h
- Cálculo de mínimo, máximo e média
- Verificação de CRC nas leituras dos sensores
- Indicação de leitura indisponível/desatualizada
- Controle de brilho da tela
- Apagamento automático do display
- Gesto de dois toques para apagar e reativar a tela
- Registro opcional das medições em CSV no microSD
- Histórico em RAM mesmo sem cartão SD
- Testes de lógica para modelo e gerenciamento de energia da tela

---

## Controle PID

O estudo acadêmico associado ao projeto também aborda o **controle PID**.

Um controlador PID utiliza três componentes:

- **P — Proporcional:** reage ao erro atual;
- **I — Integral:** considera o erro acumulado;
- **D — Derivativo:** considera a velocidade de variação do erro.

A apresentação utilizada pelo grupo está disponível em:

[`docs/Apresentacao_Controle_PID.pptx`](docs/Apresentacao_Controle_PID.pptx)

Nesta versão do firmware, o PID é parte do **estudo e da evolução proposta**, não um recurso já conectado ao ar-condicionado.

---

## Arquitetura do software

```text
Sensores SCD41 / SHT45
        │
        ▼
   sensors.c
        │
        ▼
     Queue
        │
        ▼
   app_main.c ─────► model.c ─────► Histórico em RAM
        │                │
        │                └────────► storage.c ─► microSD / CSV
        │
        ▼
      ui.c
        │
        ▼
   LVGL + Display
```

### Principais arquivos

| Arquivo | Responsabilidade |
|---|---|
| `main/app_main.c` | Coordenação principal das tarefas |
| `main/sensors.c` | Leitura do SCD41 e SHT45 |
| `main/model.c` | Conversões, validação e histórico |
| `main/ui.c` | Interface gráfica em LVGL |
| `main/board.c` | Display, touch, I²C e hardware |
| `main/screen_power.c` | Brilho e gerenciamento da tela |
| `main/storage.c` | Registro opcional em microSD |

---

## Tecnologias

- C
- ESP-IDF 5.4.0
- PlatformIO
- FreeRTOS
- LVGL 8.3.11
- I²C
- SPI
- CSV

---

## Como compilar

### Pré-requisitos

- Visual Studio Code
- Extensão PlatformIO IDE
- Cabo USB de dados

### Passos

1. Clone este repositório.
2. Abra a pasta do projeto no Visual Studio Code.
3. Aguarde o PlatformIO preparar o ambiente.
4. Execute **Build** para compilar.
5. Conecte o ESP32-S3.
6. Execute **Upload** para gravar o firmware.
7. Use o monitor serial em `115200 baud` para diagnóstico.

Pelo terminal:

```powershell
pio run
pio run -t upload --upload-port COM5
pio device monitor --port COM5 --baud 115200
```

Substitua `COM5` pela porta correspondente ao seu dispositivo.

---

## Dados registrados

Quando um cartão microSD FAT32 está disponível, o projeto pode registrar:

```csv
uptime_s,co2_ppm,temperature_c,humidity_pct
```

A versão atual utiliza tempo desde a inicialização (`uptime_s`) e ainda não implementa relógio/calendário.

---

## Próximas evoluções

- Definição de limites de temperatura configuráveis
- Alarme visual e sonoro
- Comunicação Wi-Fi
- Dashboard remoto
- Envio de notificações aos responsáveis
- Registro com data e hora reais
- Integração com sistema de climatização
- Verificação de funcionamento do ar-condicionado
- Estratégia de atuação segura em caso de falha
- Implementação e validação de malha de controle PID, caso adequada ao atuador disponível

---

## Estrutura do repositório

```text
EstacaoAmbiental/
├── boards/
├── components/
├── docs/
│   ├── Apresentacao_Controle_PID.pptx
│   ├── Origem_da_Ideia.png
│   └── README_TECNICO_ORIGINAL.md
├── firmware/
├── main/
├── tests/
├── .gitignore
├── CMakeLists.txt
├── platformio.ini
├── sdkconfig.defaults
└── README.md
```

---

## Integrantes

- Caio Watari
- Davi Luca Santos
- Lucas Kenji
- Mariana Caires
- Yan Diego

---

## Contexto acadêmico

Projeto desenvolvido para fins acadêmicos, envolvendo conceitos de:

- Sistemas embarcados
- Instrumentação
- Sensoriamento ambiental
- Automação
- Controle em malha fechada
- Controle PID
- Aquisição e registro de dados

---

## Documentação adicional

O README técnico detalhado da versão original do firmware foi preservado em:

[`docs/README_TECNICO_ORIGINAL.md`](docs/README_TECNICO_ORIGINAL.md)
