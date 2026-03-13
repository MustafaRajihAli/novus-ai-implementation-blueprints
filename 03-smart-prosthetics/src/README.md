# 🦿 Code Implementation

> **Smart Prosthetics — AI-Adaptive Myoelectric Limb Platform** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---

Gesture Classifier

```json
# src/services/gesture_classifier.py"""Trains a patient-specific LSTM gesture classifier from EMGcalibration session data and exports to TensorFlow Lite fordeployment to the STM32 embedded device."""import numpy as npimport tensorflow as tffrom tensorflow.keras import layers, Modelfrom src.models.calibration import CalibrationSessionWINDOW_SIZE = 200   # 200ms at 1000Hz sampling rateN_FEATURES = 8      # 8 EMG electrode channelsN_GESTURES = 25class GestureClassifierTrainer:    def build_model(self) -> Model:        """LSTM sequence classifier for EMG gesture recognition."""        inputs = tf.keras.Input(shape=(WINDOW_SIZE, N_FEATURES))        x = layers.LSTM(128, return_sequences=True)(inputs)        x = layers.Dropout(0.3)(x)        x = layers.LSTM(64)(x)        x = layers.Dense(64, activation="relu")(x)        x = layers.Dropout(0.2)(x)        outputs = layers.Dense(N_GESTURES, activation="softmax")(x)        model = Model(inputs, outputs)        model.compile(            optimizer="adam",            loss="sparse_categorical_crossentropy",            metrics=["accuracy"]        )        return model    def train_patient_model(self, session: CalibrationSession) -> str:        """        Train and export a patient-specific model from calibration data.        Returns the path to the exported .tflite model file.        """        X, y = self._prepare_features(session)        model = self.build_model()        model.fit(X, y, epochs=50, batch_size=32, validation_split=0.2,                  callbacks=[tf.keras.callbacks.EarlyStopping(patience=8)])        # Convert to TensorFlow Lite for on-device inference        converter = tf.lite.TFLiteConverter.from_keras_model(model)        converter.optimizations = [tf.lite.Optimize.DEFAULT]        tflite_model = converter.convert()        path = f"models/patients/{session.patient_id}_v{session.version}.tflite"        with open(path, "wb") as f:            f.write(tflite_model)        return path    def _prepare_features(self, session: CalibrationSession):        """        Segment raw EMG recordings into overlapping windows,        apply bandpass filter and RMS feature extraction.        """        windows, labels = [], []        for gesture_id, recordings in session.gesture_recordings.items():            for rec in recordings:                filtered = self._bandpass(np.array(rec), 20, 500, 1000)                for i in range(0, len(filtered) - WINDOW_SIZE, WINDOW_SIZE // 2):                    windows.append(filtered[i:i + WINDOW_SIZE])                    labels.append(gesture_id)        return np.array(windows), np.array(labels)    def _bandpass(self, signal, low, high, fs):        """Simple bandpass using scipy. Production uses CMSIS-DSP on-device."""        from scipy.signal import butter, filtfilt        nyq = fs / 2        b, a = butter(4, [low/nyq, high/nyq], btype="band")        return filtfilt(b, a, signal, axis=0)
```

Embedded Firmware

```cpp
/* src/firmware/emg_acquisition.c * Runs on STM32H7. Acquires 8-channel EMG at 1000Hz, * preprocesses via CMSIS-DSP, runs TFLite gesture classifier, * and drives motor controller PWM outputs. */#include "stm32h7xx_hal.h"#include "arm_math.h"#include "tensorflow/lite/micro/micro_interpreter.h"#define EMG_CHANNELS     8#define WINDOW_SAMPLES   200#define MOTOR_CHANNELS   5static float32_t emg_buffer[EMG_CHANNELS][WINDOW_SAMPLES];static uint16_t  buf_index = 0;/* Called by ADC DMA interrupt every 1ms */void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef *hadc) {    for (int ch = 0; ch < EMG_CHANNELS; ch++) {        emg_buffer[ch][buf_index] = read_emg_channel(ch);    }    buf_index++;    if (buf_index >= WINDOW_SAMPLES) {        buf_index = 0;        classify_and_actuate();    }}void classify_and_actuate(void) {    /* Apply CMSIS-DSP bandpass filter per channel */    for (int ch = 0; ch < EMG_CHANNELS; ch++) {        arm_biquad_cascade_df2T_f32(&bandpass_filters[ch],                                     emg_buffer[ch],                                     filtered_buffer[ch],                                     WINDOW_SAMPLES);    }    /* Copy to TFLite input tensor */    float* input = tflite_interpreter->input(0)->data.f;    for (int s = 0; s < WINDOW_SAMPLES; s++)        for (int ch = 0; ch < EMG_CHANNELS; ch++)            input[s * EMG_CHANNELS + ch] = filtered_buffer[ch][s];    /* Run inference (< 8ms on Cortex-M7 @ 480MHz) */    tflite_interpreter->Invoke();    /* Get winning gesture class */    float* output = tflite_interpreter->output(0)->data.f;    int gesture = argmax(output, N_GESTURES);    float confidence = output[gesture];    if (confidence > 0.85f) {        set_motor_targets(gesture_to_motor_map[gesture]);    }    /* Stream telemetry over BLE */    ble_send_telemetry(gesture, confidence, emg_rms_all_channels());}
```

Dockerfile

```dockerfile
FROM python:3.11-slimWORKDIR /appRUN apt-get update && apt-get install -y \    libhdf5-dev pkg-config \    && rm -rf /var/lib/apt/lists/*COPY requirements.txt .RUN pip install --no-cache-dir -r requirements.txtCOPY . .EXPOSE 8000CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

docker-compose.yml

```yaml
version: "3.9"services:  api:    build: .    ports: ["8000:8000"]    env_file: .env    depends_on: [postgres, redis]  postgres:    image: postgres:15-alpine    environment:      POSTGRES_DB: ${DB_NAME}      POSTGRES_USER: ${DB_USER}      POSTGRES_PASSWORD: ${DB_PASSWORD}    volumes: [pg_data:/var/lib/postgresql/data]  redis:    image: redis:7-alpine  model-trainer:    build: .    command: python -m src.workers.model_trainer    env_file: .env    depends_on: [postgres]  ota-server:    image: minio/minio    command: server /data    environment:      MINIO_ROOT_USER: ${MINIO_USER}      MINIO_ROOT_PASSWORD: ${MINIO_PASSWORD}    ports: ["9000:9000", "9001:9001"]    volumes: [ota_data:/data]volumes:  pg_data:  ota_data:
```

.env.example
DB_NAME=prosthetics_platformDB_USER=novus_adminDB_PASSWORD=change_in_productionREDIS_URL=redis://redis:6379/0AWS_REGION=me-south-1S3_BUCKET_MODELS=novus-prosthetics-modelsS3_BUCKET_SESSIONS=novus-prosthetics-sessionsEMR_FHIR_BASE_URL=https://fhir.hospital.iq/R4EMR_CLIENT_ID=EMR_CLIENT_SECRET=MINIO_USER=novus_otaMINIO_PASSWORD=change_in_productionSENTRY_DSN=

---

*[← Back to Smart Prosthetics](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
