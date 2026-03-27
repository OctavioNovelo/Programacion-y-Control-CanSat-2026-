# CanSat 2026: Stereo Vision & Telemetry System (ModelEggs)

Este repositorio contiene la arquitectura de software, especificaciones de hardware y protocolos de comunicación para la misión CanSat 2026 del equipo ModelEggs. El sistema destaca por su capacidad de transmisión mediante conmutación dinámica de modulación de radio frecuencia.

## Hardware
- **OBC:** STM32F401CCU6 (Blackpill).
- **Sensores:** IMU BNO085, BME280.
- **Carga Útil:** 2x ESP32-CAM (OV2640).
- **Radio:** Semtech SX1278 (433MHz).
- **Ground Station:** Antena Yagi-Uda de 5 elementos + Receptor STM32.

## 📡 Protocolo de Comunicaciones (Link Layer)
Para garantizar la integridad de la imagen en el descenso, se utiliza un protocolo de tramas de longitud fija sobre GFSK:

| Campo | Tamaño | Función |
| :--- | :--- | :--- |
| Header | 2 Bytes | Sincronización de trama (0xAA55) |
| SEQ ID | 1 Byte | Reordenamiento de paquetes |
| Payload | 124 Bytes | Datos binarios de imagen (L8 Grayscale) |
| Checksum | 1 Byte | Validación CRC-8 |

### Estrategia de Conmutación
1. **Modo Telemetría (LoRa):** Envío continuo de estado de sensores.
2. **Sync Command:** A los 410m de altitud, se envía un comando de pre-alerta.
3. **Modo Imagen (GFSK):** Conmutación a 50kbps para descarga de carga útil a los 400m.

## 💻 Procesamiento de Imagen (C# Engine)
El software de tierra está desarrollado en **.NET Core**, optimizado para baja latencia:
- **Optimización:** Uso de `SixLabors.ImageSharp` con punteros de memoria y `Span<T>` para evitar recolección de basura (GC) durante la fusión.
- **Fusión Anaglifo:** - Canal R: Imagen Izquierda.
  - Canales G+B: Imagen Derecha.
- **Corrección de Actitud:** Integración de cuaterniones de telemetría para compensar la inclinación del satélite durante la captura.

## 📊 Link Budget y Tiempos
- **Sensibilidad RX:** -110 dBm (GFSK).
- **Tamaño de Imagen:** ~35 KB (L8 Grayscale).
- **Airtime estimado:** 6.1 segundos a 50 kbps.
- **Margen de Seguridad:** Basado en una velocidad de descenso de 6 m/s, el sistema completa la descarga 12 segundos antes del impacto.

## 📁 Estructura del Repo
- `/Firmware_OBC`: Código en C++/STM32CubeIDE.
- `/Firmware_CAM`: Captura y fragmentación SPI para ESP32-CAM.
- `/Ground_Station`: Script de procesamiento C# y UI de telemetría.
- `/Docs`: Cálculos detallados de Link Budget y esquemáticos.