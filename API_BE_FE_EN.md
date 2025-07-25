# Backend-Frontend API List for ESP32 Signal Generator

## WebSocket API (/ws)

| Variable/Command | Explanation | Access Mode |
|------------------|-------------|------------|
| `getStatus` | Gets the current system status (CTCSS, DCS, Burst, Calibration) | Frontend → Backend |
| `setCtcss` | Configures CTCSS parameters (enable, frequency, level) | Frontend → Backend |
| `setDcs` | Configures DCS parameters (enable, code, inverted, level) | Frontend → Backend |
| `setDcsBurst` | Configures DCS Burst parameters (enable, burstTime, pauseTime, sequences, useTurnOffCode) | Frontend → Backend |
| `startDcsTelegram` | Initiates a DCS telegram transmission | Frontend → Backend |
| `stopDcs` | Stops DCS transmission | Frontend → Backend |
| `setGpsCalibration` | Enables/disables GPS calibration | Frontend → Backend |
| `forceCalibration` | Forces manual calibration | Frontend → Backend |
| `resetCalibration` | Resets calibration data | Frontend → Backend |
| `setDtmf` | Configures DTMF parameters | Frontend → Backend |
| `playDtmfTone` | Plays a specific DTMF tone | Frontend → Backend |
| `stopDtmfTone` | Stops the current DTMF tone | Frontend → Backend |
| `playDtmfSequence` | Plays a DTMF sequence | Frontend → Backend |
| `stopDtmfSequence` | Stops the current DTMF sequence | Frontend → Backend |

## Backend → Frontend Notifications

| Variable | Explanation | Access Mode |
|----------|-------------|------------|
| `status` | General system status update | Backend → Frontend |
| `dcsBurstStatus` | DCS Burst status update (status, currentSequence, totalSequences, inProgress) | Backend → Frontend |
| `calibration` | Calibration status update (enabled, status, factor, progress, confidence, ppsActive) | Backend → Frontend |
| `dtmfSequenceComplete` | DTMF sequence completion notification | Backend → Frontend |
| `dtmfToneComplete` | Individual DTMF tone completion notification | Backend → Frontend |
| `dtmfRepeatRestart` | DTMF repeat cycle restart notification | Backend → Frontend |

## Main State Variables

| Variable | Explanation | Access Mode |
|----------|-------------|------------|
| `gBurstConfig` | Global Burst configuration (dcsBurstSequences, burstOnTime, burstOffTime, useTurnOffCode, burstEnabled, etc.) | Bidirectional access |
| `dtmfState` | DTMF state (active, currentDigit, level, sequence, sequenceActive, repeatMode, etc.) | Bidirectional access |
| `CalibrationData` | Calibration data (enabled, status, factor, progress, confidence) | Bidirectional access |

## Status Indicators

| Variable | Explanation | Access Mode |
|----------|-------------|------------|
| `dtmfSequenceCompleteFlag` | Flag for DTMF sequence completion | ISR → Frontend |
| `dtmfToneCompleteFlag` | Flag for DTMF tone completion | ISR → Frontend |
| `dtmfRepeatRestartFlag` | Flag for DTMF cycle restart | ISR → Frontend |
| `burstSequenceComplete` | Flag for Burst sequence completion | ISR → Frontend |

## API Syntax Examples

### Frontend to Backend Commands

#### Get Status
```json
{
  "command": "getStatus"
}
```

#### Set CTCSS
```json
{
  "command": "setCtcss",
  "enable": true,
  "frequency": 100.0,
  "level": 128
}
```

#### Set DCS
```json
{
  "command": "setDcs",
  "enable": true,
  "code": 23,
  "inverted": false,
  "level": 128
}
```

#### Set DCS Burst
```json
{
  "command": "setDcsBurst",
  "enable": true,
  "burstTime": 500,
  "pauseTime": 500,
  "sequences": 3,
  "useTurnOffCode": true
}
```

#### Start DCS Telegram
```json
{
  "command": "startDcsTelegram",
  "code": 23,
  "sequences": 3,
  "burstTime": 500,
  "pauseTime": 500,
  "useTurnOffCode": true
}
```

#### Set GPS Calibration
```json
{
  "command": "setGpsCalibration",
  "enable": true
}
```

#### Play DTMF Tone
```json
{
  "command": "playDtmfTone",
  "digit": "5",
  "duration": 100
}
```

#### Play DTMF Sequence
```json
{
  "command": "playDtmfSequence",
  "sequence": "12345",
  "toneDuration": 100,
  "pauseDuration": 50,
  "repeat": false
}
```

### Backend to Frontend Responses

#### Status Update
```json
{
  "status": "ok",
  "ctcss": {
    "enabled": true,
    "frequency": 100.0
  },
  "dcs": {
    "enabled": false,
    "code": "023",
    "inverted": false
  },
  "burst": {
    "enabled": false,
    "burstTime": 500,
    "pauseTime": 500,
    "dcsSequences": 3,
    "currentSequence": 0,
    "sequenceComplete": false,
    "useTurnOffCode": true,
    "sendingTurnOffCode": false
  }
}
```

#### DCS Burst Status
```json
{
  "type": "dcsBurstStatus",
  "status": "sequence_complete",
  "currentSequence": 2,
  "totalSequences": 3,
  "inProgress": true
}
```

#### Calibration Update
```json
{
  "status": "ok",
  "calibration": {
    "enabled": true,
    "status": 2,
    "factor": 1.000023,
    "progress": 75.5,
    "confidence": 98.2,
    "ppsActive": true
  }
}
```

#### DTMF Status Update
```json
{
  "type": "dtmfStatus",
  "sequenceActive": true,
  "currentDigit": "5",
  "sequenceIndex": 2,
  "sequenceLength": 5,
  "repeatMode": false
}
```
