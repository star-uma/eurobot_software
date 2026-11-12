# 08 - Estado de avance del proyecto

Última actualización: Fase 1 (documentación SDD inicial).

## 1. Qué funciona

| Funcionalidad | Estado | Comentario |
|---------------|--------|------------|
| Simulación Gazebo + robot spawn | ✅ | `sim.launch.py` lanza mundo, robot y bridges |
| Odometría y TF en simulación | ✅ | `diff_cont` publica `/odom` y TF `odom→base_link` |
| Visión ArUco | ✅ | `vision_node` detecta marcadores 4X4_50 en `/camera/image_raw` |
| ros2_control (diff_cont, joint_broad, actuator_cont) | ✅ | Controladores cargan y spawnean |
| Parámetros centralizados | ✅ | `parametros_robot.yaml` leído por URDF y sim.launch.py |
| Teleop por teclado | ✅ | `cmd_vel_timeout: 0.25` detiene el robot si no llegan comandos |

## 2. Qué está a medio hacer o roto

| Funcionalidad | Estado | Problema conocido |
|---------------|--------|-------------------|
| Actuador prismático | ⚠️ Parcial | Mueve la caja, ajustar PID y modelarlo correctamente, solo hay un pilar de prueba.|
| Filtro lidar | ⚠️ Parcial | `filtro_lidar` publica `/scan_filtered` (máx 1.0 m) - Falta probarlo en gazebo -|
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
| 2 | [Control Xbox](09-especificacion-xbox.md) (paquete `eurobot_software_xbox`) | 🔄 En curso (paquete creado, pendiente colcon build y pruebas) |
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