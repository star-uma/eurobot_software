# 08 - Estado de avance del proyecto

Última actualización: teleop Xbox funcionando (movimiento + actuador con flechas).

## 1. Qué funciona

| Funcionalidad | Estado | Comentario |
|---------------|--------|------------|
| Simulación Gazebo + robot spawn | ✅ | `sim.launch.py` lanza mundo, robot y bridges |
| Odometría y TF en simulación | ✅ | `diff_cont` publica `/diff_cont/odom` y TF `odom→base_link` |
| Visión ArUco | ✅ | `vision_node` detecta marcadores 4X4_50 en `/camera/image_raw` |
| ros2_control (diff_cont, joint_broad, actuator_cont) | ✅ | Controladores cargan y spawnean |
| Parámetros centralizados | ✅ | `parametros_robot.yaml` leído por URDF y sim.launch.py |
| Teleop por teclado | ✅ | `cmd_vel_timeout: 0.25` detiene el robot si no llegan comandos |
| Teleop por mando Xbox | ✅ | Movimiento (RB + stick izq.) y actuador con flechas (`eurobot_software_xbox`) |

## 2. Qué está a medio hacer o roto

| Funcionalidad | Estado | Problema conocido |
|---------------|--------|-------------------|
| Actuador prismático | ⚠️ Parcial | Controlable con las flechas del mando (0–0.38 m); falta ajustar PID, modelado y posición correcta para la ventosa |
| Filtro lidar | ⚠️ Prototipo no integrado | Código probado aparte por un compañero; sin entrypoint en `setup.py`, no se lanza en ROS 2 |
| Botón A / ventosa | ❌ No hecho | `suction_cup_link` es un link fijo sin plugin de agarre; falta mecanismo en Gazebo y electroválvula en el real |
| Robot real (ESP32) | ❌ No hecho | Firmware básico siguelíneas; sin puente ROS 2 ni ros2_control |
| Publicar poses ArUco | ❌ No hecho | `vision_node` solo muestra en ventana, no publica topic con poses |
| Mapa 2D para Nav2 | ❌ No hecho | Pendiente crear archivo .yaml + imagen del escenario |
| Sensores inferiores | ❌ No hecho | Solo lidar superior existe; pendiente añadir sensores de suelo/rival |

## 3. Bugs conocidos

—

## 4. Próximas prioridades

| Prioridad | Tarea | Estado |
|-----------|-------|--------|
| 1 | Documentación SDD (esta fase) | 🔄 En curso |
| 2 | [Control Xbox](09-especificacion-xbox.md) (paquete `eurobot_software_xbox`) | ✅ Funciona (movimiento + actuador); pendiente botón A / ventosa |
| 3 | [Actuador real](10-especificacion-actuador-real.md) (pilares + barra rotativa + ventosas) | Pendiente |
| 4 | Nav2 + odometría | Pendiente |
| 5 | Detección ArUco para centrado de caja | Pendiente |
| 6 | Sensores rivales/objetos (inferiores) | Pendiente |
| 7 | FSM de competición final | Pendiente |
| 8 | Perfeccionamiento iterativo | Pendiente |

## 5. Notas para la IA

- No tocar `parametros_robot.yaml` sin actualizar también `controllers.yaml` y los URDF afectados.
- `sim.launch.py` genera un YAML temporal en `/tmp/`; no editar ese archivo directamente.
- Los topics de cámara y lidar vienen de Gazebo; en el robot real habrá que sustituir por hardware real.
- El firmware ESP32 es independiente (PlatformIO) y aún no se comunica con ROS 2.

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Arquitectura general](01-arquitectura-general.md)
- [Guía de onboarding](07-guia-onboarding-companeros.md)
- [Plantilla de especificación](plantilla-especificacion.md)