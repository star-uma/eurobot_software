# 01 - Arquitectura general del software Eurobot 2027

## 1. Meta-repositorio y sub-repositorios

El repositorio raíz `eurobot_software` es un meta-repositorio que orquesta 4 sub-repositorios mediante `vcs import` (archivo `robot.repos`). Cada sub-repo contiene uno o más paquetes ROS 2.

| Sub-repositorio                                                                        | Paquete(s) ROS 2                                      | Propósito                              |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------- | -------------------------------------- |
| [eurobot_software_comun](github.com/star-uma/eurobot_software_comun)                   | eurobot_software_description, eurobot_software_vision, eurobot_software_xbox | Código común a robot real y simulación |
| [eurobot_software_real](github.com/star-uma/eurobot_software_real)                     | eurobot_software_real                                 | Parámetros del robot real              |
| [eurobot_software_simulacion](github.com/star-uma/eurobot_software_simulacion)         | eurobot_software_simulacion                           | Mundos Gazebo y launch de simulación   |
| [eurobot_software_firmware_esp32](github.com/star-uma/eurobot_software_firmware_esp32) | PlatformIO/Arduino                               | Firmware de bajo nivel ESP32           |
```text
Nota: PlatformIO no es un paquete de ros2, está dentro de la estructura pero en la raíz del repositorio se encuentra el archivo vación COLCON_IGNORE, por tanto colcon no lo compilará como el resto de paquetes.
```
## 2. Mapa de paquetes

### eurobot_software_description
Contiene la descripción del robot (URDF/Xacro), configuración y launch de estado del robot.

- `urdf/robot.urdf.xacro` — Ensamblador principal del robot. Incluye robot_core, lidar, camara, actuador, ros2_control.
- `urdf/robot_core.xacro` — Chasis, ruedas diferenciales, rueda loca, base_link.
- `urdf/lidar.xacro` — Sensor GPU lidar en Gazebo (`/scan`).
- `urdf/camara.xacro` — Cámara en Gazebo (`/camera/image_raw`) con corrección óptica OpenCV.
- `urdf/actuador.xacro` — Torre fija, carro prismático (elevator_joint), cabezal de ventosas.
- `urdf/ros2_control.xacro` — Definición de interfaces ros2_control.
- `urdf/inertial_macros.xacro` — Macros de inercia reutilizables.
- `config/parametros_robot.yaml` — Fuente única de verdad de parámetros físicos (base, actuator).
- `config/controllers.yaml` — Configuración de controladores ros2_control (diff_cont, joint_broad, actuator_cont).
- `config/test_controllers.yaml` — Configuraciones adicionales (pendiente revisar uso).
- `launch/rsp.launch.py` — Publica `robot_description` vía xacro, acepta `use_sim`, `use_sim_time`, `controllers_yaml`.

### eurobot_software_xbox
- `config/joy_teleop.yaml` — Configuración de mapeo del mando Xbox (movido desde `eurobot_software_description`).
- `launch/xbox.launch.py` — Lanza `joy_node` + `xbox_teleop_node`; acepta `use_sim_time`.
- `eurobot_software_xbox/xbox_teleop.py` — Nodo `xbox_teleop_node`. Suscribe `/joy` (`sensor_msgs/Joy`) y `/joint_states` (`sensor_msgs/JointState`); publica `/cmd_vel` (`geometry_msgs/Twist`) y `/actuator_cont/joint_trajectory` (`trajectory_msgs/JointTrajectory`).

### eurobot_software_vision
- `eurobot_software_vision/vision_node.py` — Nodo `vision_node`. Suscribe `/camera/image_raw`, detecta marcadores ArUco (diccionario 4X4_50), publica visualización con `cv2.imshow`.
- `eurobot_software_vision/filtro_lidar.py` — Prototipo no integrado: suscribe `/scan`, filtra rangos > 1.0 m y publica `/scan_filtered`. **No tiene entrypoint en `setup.py`**, por lo que no se puede lanzar con `ros2 run` actualmente.

### eurobot_software_simulacion
- `worlds/eurobot.sdf` — Mundo principal de Gazebo Ignition.
- `worlds/caja.sdf` — Modelo de caja (rotadas horizontalmente, pose pi/2).
- `worlds/materials/textures/` — Texturas (mapa_eurobot.png, aruco_13.png).
- `launch/sim.launch.py` — Lance completo de simulación: Gazebo, rsp, spawn del robot, bridge ROS↔Gazebo, static_tf, spawners de controladores.
- `launch/test.launch.py` — Launch de pruebas (pendiente detallar).

### eurobot_software_real
- Paquete para parámetros específicos del robot real (pendiente detallar contenido exacto).

### eurobot_software_firmware_esp32 (SIMAS)
- `SIMAS/src/main.cpp` — Arduino/ESP32. Controla 2 servos (motores pines 18, 19) y lee 2 sensores IR siguelíneas (pines 32, 33). Comportamiento: siguelíneas durante 10 s, luego maniobra final y paro.

## 3. Topics principales

| Topic             | Tipo                  | Publicador                       | Suscriptor   | Descripción                  |
| ----------------- | --------------------- | -------------------------------- | ------------ | ---------------------------- |
| /camera/image_raw | sensor_msgs/Image     | Gazebo (cámara)                  | vision_node  | Imagen de la cámara          |
| /scan             | sensor_msgs/LaserScan | Gazebo (lidar)                   | — (filtro_lidar no integrado) | Escaneo lidar crudo |
| /scan_filtered    | sensor_msgs/LaserScan | filtro_lidar (prototipo)         | —            | Escaneo filtrado (máx 1.0 m) |
| /joy              | sensor_msgs/Joy       | joy_node (driver mando)          | xbox_teleop_node | Entrada del mando Xbox   |
| /cmd_vel          | geometry_msgs/Twist   | teleop / xbox_teleop_node        | diff_cont    | Comando de velocidad         |
| /actuator_cont/joint_trajectory | trajectory_msgs/JointTrajectory | xbox_teleop_node | actuator_cont | Objetivo de posición del actuador |
| /joint_states     | sensor_msgs/JointState | joint_broad                     | xbox_teleop_node | Posiciones de las articulaciones |
| /dynamic_joint_states | control_msgs/DynamicJointState | controller_manager | —     | Estados dinámicos de ros2_control |
| /robot_description | std_msgs/String      | robot_state_publisher            | —            | Descripción URDF del robot   |
| /diff_cont/odom   | nav_msgs/Odometry     | diff_cont                        | —            | Odometría real (~18 Hz)      |
| /odom             | nav_msgs/Odometry     | ros_gz_bridge                    | —            | Bridge de Gazebo (sin datos) |
| /tf / /tf_static  | tf2_msgs/TFMessage    | diff_cont, robot_state_publisher, ros_gz_bridge | — | Árbol de transformaciones |
| /clock            | rosgraph_msgs/Clock   | Gazebo (bridge)                  | diff_cont    | Reloj de simulación          |

## 4. Nodos principales

| Nodo                  | Paquete                 | Executable                 | Función                           |
| --------------------- | ----------------------- | -------------------------- | --------------------------------- |
| vision_node           | eurobot_software_vision | vision_node                | Detección ArUco en imagen         |
| lidar_filter_node     | eurobot_software_vision | — (sin entrypoint)         | Filtro de rango lidar (prototipo) |
| xbox_teleop_node      | eurobot_software_xbox   | xbox_teleop_node           | Lee /joy y publica /cmd_vel + trayectoria del actuador |
| robot_state_publisher | robot_state_publisher   | robot_state_publisher      | Publica TF y robot_description    |
| joint_broad           | controller_manager      | spawner                    | Publica estados de articulaciones |
| diff_cont             | controller_manager      | spawner                    | Controlador tracción diferencial  |
| actuator_cont         | controller_manager      | spawner                    | Controlador trayectoria actuador  |
| static_tf             | tf2_ros                 | static_transform_publisher | TF provisional map→odom           |
| create                | ros_gz_sim              | create                     | Spawnea entidad en Gazebo         |
| parameter_bridge      | ros_gz_bridge           | parameter_bridge           | Puente ROS↔Gazebo                 |

## 5. Launch files principales

| Archivo        | Paquete                      | Propósito                                                  |
| -------------- | ---------------------------- | ---------------------------------------------------------- |
| sim.launch.py  | eurobot_software_simulacion  | Lanza Gazebo, robot, bridges y controladores en simulación |
| rsp.launch.py  | eurobot_software_description | Publica descripción del robot (URDF vía xacro)             |
| test.launch.py | eurobot_software_simulacion  | Launch de pruebas (pendiente detallar)                     |
| xbox.launch.py | eurobot_software_xbox        | Lanza `joy_node` + `xbox_teleop_node` (acepta `use_sim_time`) |

## 6. Flujo de datos típico (simulación)

1. `sim.launch.py` inicia Gazebo con `eurobot.sdf`.
2. Gazebo publica `/camera/image_raw`, `/scan`, `/odom`, `/clock`, `/tf`.
3. `rsp.launch.py` genera `robot_description` a partir de xacro + `parametros_robot.yaml` + `controllers.yaml`.
4. `parameter_bridge` traduce temas Gazebo ↔ ROS 2.
5. `diff_cont` recibe `/cmd_vel` y mueve las ruedas en Gazebo; publica `/diff_cont/odom` y `/tf`.
6. `vision_node` procesa `/camera/image_raw` y detecta ArUco.
7. `filtro_lidar` procesa `/scan` y publica `/scan_filtered`.
8. `static_tf` publica `map → odom` (provisional).

## 7. Sim-to-real

La estrategia es usar `ros2_control` para unificar simulación y robot real, cambiando solo el backend (Gazebo vs hardware ESP32). Los parámetros físicos viven en `parametros_robot.yaml` y se inyectan tanto en el URDF como en `controllers.yaml` vía `sim.launch.py`.

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Especificación de hardware](02-especificacion-hardware.md)
- [Especificación de simulación](03-especificacion-simulacion.md)
- [Especificación de robot real](04-especificacion-robot-real.md)
- [Especificación de visión](05-especificacion-vision.md)
- [Especificación de ros2_control](06-especificacion-ros2-control.md)
- [Estado de avance](08-estado-avance.md)