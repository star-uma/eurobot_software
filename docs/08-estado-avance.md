# 08 - Estado de avance del proyecto

Última actualización: Fase 1 (documentación SDD inicial).

## 1. Qué funciona

| Funcionalidad | Estado | Comentario |
|---------------|--------|------------|
| Workspace compila | ✅ | `colcon build --symlink-install` ejecutado con éxito |
| Simulación Gazebo + robot spawn | ✅ | `sim.launch.py` lanza mundo, robot y bridges |
| Odometría y TF en simulación | ✅ | `diff_cont` publica `/odom` y TF `odom→base_link` |
| Visión ArUco | ✅ | `vision_node` detecta marcadores 4X4_50 en `/camera/image_raw` |
| Filtro lidar | ✅ | `filtro_lidar` publica `/scan_filtered` (máx 1.0 m) |
| ros2_control (diff_cont, joint_broad, actuator_cont) | ✅ | Controladores cargan y spawnean |
| Parámetros centralizados | ✅ | `parametros_robot.yaml` leído por URDF y sim.launch.py |

## 2. Qué está a medio hacer o roto

| Funcionalidad | Estado | Problema conocido |
|---------------|--------|-------------------|
| Actuador prismático | ⚠️ Parcial | Ganancia insuficiente (log 5_11_26); fuerza no mueve la caja |
| static_tf map→odom | ⚠️ Provisional | Nodo temporal; debe sustituirse por odometría fiable + Nav2 |
| Teleop por teclado | ⚠️ A pulsos | `cmd_vel_timeout: 0.25` evita avance infinito, pero no es control continuo |
| Robot real (ESP32) | ❌ No integrado | Firmware básico siguelíneas; sin puente ROS 2 ni ros2_control |
| Publicar poses ArUco | ❌ No hecho | `vision_node` solo muestra en ventana, no publica topic con poses |
| Mapa 2D para Nav2 | ❌ No hecho | Pendiente crear archivo .yaml + imagen del escenario |
| Sensores inferiores | ❌ No hecho | Solo lidar superior existe; pendiente añadir sensores de suelo/rival |

## 3. Bugs conocidos

1. **Actuador sin fuerza:** La articulación `elevator_joint` no mueve la carga. Se reportó ganancia a 0.1 en los logs. Revisar PID y límites de effort en `controllers.yaml` y `ros2_control.xacro`.
2. **static_tf provisional:** `map→odom` es estático; en movimiento real el frame se desplace correctamente solo si se sustituye por odometría.
3. **Cajas rotadas:** Las cajas en `caja.sdf` ya se rotaron pi/2 (log 4_11_26), pero verificar que colisionan correctamente con el actuador.

## 4. Próximas prioridades

| Prioridad | Tarea | Estado |
|-----------|-------|--------|
| 1 | Documentación SDD (esta fase) | 🔄 En curso |
| 2 | [Control Xbox](09-especificacion-xbox.md) (paquete `eurobot_software_xbox`) | Pendiente |
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