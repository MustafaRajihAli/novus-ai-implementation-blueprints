# 🦿 System Architecture

> **Smart Prosthetics — AI-Adaptive Myoelectric Limb Platform** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---

Architecture Flow:
Patient Residual Muscle → EMG Electrode ArrayEMG Electrode Array → STM32 Microcontroller (embedded)STM32 → EMG Preprocessing (bandpass filter, feature extraction)STM32 → LSTM Gesture Classifier (TensorFlow Lite, on-device)LSTM Output → Motor Controller → Micro-Actuators (physical movement)Physical Contact → Pressure/Tactile SensorsPressure Sensors → Haptic Feedback Driver → LRA Motors → Patient SkinSTM32 → BLE Module → Patient Mobile AppPatient Mobile App → FastAPI Backend (4G/WiFi)FastAPI Backend → PostgreSQL (patient records, telemetry)FastAPI Backend → Therapist Dashboard (React)Nightly → OTA Model Update Service → Device Fleet

---

*[← Back to Smart Prosthetics](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
