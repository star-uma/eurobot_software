# 04 - Especificación de robot real

## 1. Paquete: eurobot_software_real

Contiene parámetros específicos del robot real. El contenido exacto del paquete debe confirmarse (pendiente de revisar archivos en `eurobot_ws/src/eurobot_software_real/`).

## 2. Firmware ESP32 (SIMAS)

Ubicación: `eurobot_ws/src/eurobot_software_firmware_esp32/SIMAS/src/main.cpp`

**Hardware:**
- 2 servos de motor: pines 18 (izquierdo) y 19 (derecho).
- 2 sensores infrarrojos siguelíneas: pines 32 (izquierdo) y 33 (derecho).
- Lectura digital: HIGH = línea negra, LOW = fondo blanco (según comentario del código).

**Comportamiento actual:**
1. **Estado 1 — Siguelíneas (primeros 10 s):** Lee sensores IR y ajusta velocidades de los servos:
   - Blanco/blanco → avanzar recto (izq 180, der 0).
   - Negro/blanco → girar izquierda (izq 90, der 0).
   - Blanco/negro → girar derecha (izq 180, der 90).
   - Negro/negro → avanzar recto.
2. **Estado 2 — Acción posterior (tras 10 s):** Paro 1 s, maniobra de giro, paro definitivo. Se ejecuta una sola vez (`accionFinalizada`).

**Limitaciones conocidas:**
- Es un prototipo básico; no hay integración con ROS 2 ni ros2_control en el robot real todavía.
- No hay control de actuador ni ventosas en el firmware actual.
- Los valores de servo (0, 90, 180) son empiricos y pueden requerir calibración.

## 3. Diferencias simulación vs robot real

| Aspecto | Simulación | Robot real |
|---------|------------|------------|
| Control de movimiento | Gazebo + diff_cont (ros2_control) | Firmware ESP32 (servos directos) |
| Odometría | Gazebo + diff_cont | Pendiente (encoderos no implementados) |
| Lidar | GPU lidar en Gazebo | Pendiente (hardware real) |
| Cámara | Gazebo sensor camera | Pendiente (hardware real) |
| Comunicación | ROS 2 nativo | Pendiente (micro-ROS o puente serie) |

## 4. Pendientes

- Implementar driver de motor real y puente con ROS 2 (micro-ROS o serial).
- Añadir encoders para odometría fiable en el robot real.
- Integrar control del actuador y ventosas en el firmware.

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Arquitectura general](01-arquitectura-general.md)
- [Especificación de hardware](02-especificacion-hardware.md)
- [Especificación de simulación](03-especificacion-simulacion.md)
- [Estado de avance](08-estado-avance.md)