# 03 - Especificación de simulación (Gazebo Ignition)

## 1. Entorno

- Motor: Gazebo Ignition Fortress (también referenciado como `gz`).
- Paquete: `eurobot_software_simulacion`.
- Distribución ROS 2: Humble.

## 2. Mundos y modelos

| Archivo | Ruta | Descripción |
|---------|------|-------------|
| eurobot.sdf | `worlds/eurobot.sdf` | Mundo principal de la competencia |
| caja.sdf | `worlds/caja.sdf` | Modelo de caja individual |
| mapa_eurobot.png | `worlds/materials/textures/` | Textura del suelo/mapa |
| aruco_13.png | `worlds/materials/textures/` | Textura/marcador ArUco |

**Nota (log 4_11_26):** Las cajas en `caja.sdf` se rotaron horizontalmente (pose pi/2) y se ajustaron sus coordenadas.

## 3. Launch de simulación: `sim.launch.py`

Secuencia de lanzamiento:

1. **Variables de entorno:** `IGN_GAZEBO_RESOURCE_PATH` y `GZ_SIM_RESOURCE_PATH` apuntan a `worlds/`.
2. **Fusión de parámetros:** Lee `parametros_robot.yaml` y `controllers.yaml`, inyecta `wheel_separation` y `wheel_radius` en `diff_cont`, guarda un YAML temporal en `/tmp/eurobot_controllers_generado.yaml`.
3. **rsp:** Incluye `rsp.launch.py` con `use_sim:=true`, `use_sim_time:=true`, y el YAML generado de controladores.
4. **Gazebo:** Lanza `gz_sim.launch.py` con `gz_args: -r <ruta/eurobot.sdf>`.
5. **spawn_entity:** Nodo `ros_gz_sim/create`, spawnea el robot desde topic `robot_description` en posición `x=0.1, y=1.0, z=0.1, Y=0.0`.
6. **bridge:** Nodo `ros_gz_bridge/parameter_bridge` con los siguientes temas:
   - `/clock` → `rosgraph_msgs/Clock`
   - `/scan` → `sensor_msgs/LaserScan`
   - `/camera/image_raw` → `sensor_msgs/Image`
   - `/odom` → `nav_msgs/Odometry`
   - `/tf` → `tf2_msgs/TFMessage` (`ignition.msgs.Pose_V`)
   - **Nota:** `cmd_vel` fue eliminado del bridge; se gestiona vía `diff_cont_spawner`.
7. **static_tf:** `static_transform_publisher` `0 0 0 0 0 0 map odom` (provisional).
8. **Spawners de controladores:** `joint_broad`, `diff_cont`, `actuator_cont` vía `controller_manager/spawner`.

## 4. Control en simulación

- `diff_cont` funciona en **bucle abierto** (`open_loop: true`) para evitar abortos por falta de feedback.
- `cmd_vel_timeout: 0.25` s: el teleop envía a pulsos para que el robot no avance infinitamente si deja de mandar.
- El reloj de simulación (`/clock`) se sincroniza con los cálculos de `diff_cont`.

## 5. Cómo lanzar (simulación)

```bash
# Terminal 1: simulación completa
ros2 launch eurobot_software_simulacion sim.launch.py

# Terminal 2: teleop por teclado
ros2 run teleop_twist_keyboard teleop_twist_keyboard

# Terminal 3: visión (opcional)
ros2 run eurobot_software_vision vision_node
```

## 6. Dependencias de simulación

```
ros-humble-ign-ros2-control
ros-humble-ros2-controllers
ros-humble-joint-state-broadcaster
ignition-fortress
```

## 7. Pendientes / conocidos

- El `static_tf` map→odom es provisional; se planea sustituir por odometría fiable + Nav2.
- El mundo y las texturas pueden requerir ajuste para la competición real (tres cajas, mapa de muralla).

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Arquitectura general](01-arquitectura-general.md)
- [Especificación de hardware](02-especificacion-hardware.md)
- [Especificación de ros2_control](06-especificacion-ros2-control.md)
- [Estado de avance](08-estado-avance.md)