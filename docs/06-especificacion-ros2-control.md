# 06 - Especificación de ros2_control

## 1. Visión general

El proyecto migró del plugin Gazebo `diff_drive` a **ros2_control** para unificar simulación y robot real. Los parámetros físicos viven en `parametros_robot.yaml` y se inyectan en `controllers.yaml` vía `sim.launch.py` (fusión dinámica en tiempo de ejecución).

## 2. Controladores definidos

Archivo: `config/controllers.yaml`

| Controlador | Tipo | Propósito |
|-------------|------|-----------|
| diff_cont | `diff_drive_controller/DiffDriveController` | Tracción diferencial de las ruedas |
| joint_broad | `joint_state_broadcaster/JointStateBroadcaster` | Publica estados de todas las articulaciones |
| actuator_cont | `joint_trajectory_controller/JointTrajectoryController` | Control del actuador (elevator_joint) |

## 3. Configuración diff_cont

Sección `diff_cont.ros__parameters`:

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| publish_rate | 50.0 | Hz de publicación de odometría |
| use_stamped_vel | false | Usar TwistStamped o Twist |
| open_loop | true | Bucle abierto (evita aborto sin feedback) |
| cmd_vel_timeout | 0.25 | s, timeout de comando de velocidad |
| left_wheel_names | ["left_wheel_joint"] | Joint rueda izquierda |
| right_wheel_names | ["right_wheel_joint"] | Joint rueda derecha |
| wheel_separation_multiplier | 1.0 | Multiplicador separación |
| left_wheel_radius_multiplier | 1.0 | Multiplicador radio rueda izq |
| right_wheel_radius_multiplier | 1.0 | Multiplicador radio rueda der |
| command_topic | cmd_vel | Topic de entrada |
| odom_frame_id | odom | Frame odometría |
| base_frame_id | base_link | Frame base del robot |
| enable_odom_tf | true | Publicar TF odom→base_link |

Límites de velocidad lineal:
- `linear.x.max_velocity: 1.0`, `min_velocity: -1.0` m/s
- `linear.x.max_acceleration: 1.0`, `min_acceleration: -1.0` m/s²

Límites de velocidad angular:
- `angular.z.max_velocity: 2.0`, `min_velocity: -2.0` rad/s
- `angular.z.max_acceleration: 2.0`, `min_acceleration: -2.0` rad/s²

**Nota:** `wheel_separation` y `wheel_radius` se inyectan dinámicamente desde `parametros_robot.yaml` en `sim.launch.py`.

## 4. Configuración actuator_cont

Sección `actuator_cont.ros__parameters`:

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| joints | ["elevator_joint"] | Articulación controlada |
| command_interfaces | ["effort"] | Emite comandos de esfuerzo (fuerza) |
| state_interfaces | ["position", "velocity"] | Lee posición y velocidad |
| gains.elevator_joint.p | 2000.0 | Ganancia proporcional PID |
| gains.elevator_joint.i | 10.0 | Ganancia integral |
| gains.elevator_joint.d | 50.0 | Ganancia derivativa |
| constraints.stopped_velocity_tolerance | 0.01 | m/s |
| constraints.goal_time | 1.0 | s |
| constraints.elevator_joint.goal | 0.03 | m (posición objetivo) |

**Problema conocido (log 5_11_26):** La articulación prismática no se mueve con fuerza suficiente; se sospecha de ganancia baja (0.1) o configuración de effort. Requiere ajuste de PID y límites.

## 5. Configuración joint_broad

- Tipo: `joint_state_broadcaster/JointStateBroadcaster`
- Publica `/joint_states` con todas las articulaciones del URDF.

## 6. Launch de controladores

### rsp.launch.py
- Publica `robot_description` procesando `robot.urdf.xacro` con `xacro`.
- Argumentos: `use_sim`, `use_sim_time`, `controllers_yaml`.
- Nodo: `robot_state_publisher` con `robot_description` y `use_sim_time`.

### sim.launch.py
- Lee `parametros_robot.yaml` y `controllers.yaml`.
- Fusiona `wheel_separation` y `wheel_radius` en `controllers_config`.
- Guarda YAML temporal en `/tmp/eurobot_controllers_generado.yaml`.
- Lanza `rsp.launch.py` con el YAML generado.
- Spawnea `joint_broad`, `diff_cont`, `actuator_cont` vía `controller_manager/spawner`.

## 7. Archivo ros2_control.xacro

Incluido en `robot.urdf.xacro`. Define las interfaces de hardware (ros2_control) para las articulaciones del robot. Su contenido exacto debe revisarse para confirmar qué joints exporta y qué interfaces usa (position, velocity, effort).

## 8. Pendientes / conocidos

- `diff_cont` en bucle abierto: adecuado para pruebas, pero para competición con Nav2 podría requerirse bucle cerrado con odometría real.
- Ganancia del actuador insuficiente para mover la caja; pendiente de tuning PID y verificación de límites de effort.
- `ros2_control.xacro` necesita una variable para alternar simulación vs real (pendiente, mencionado en log 21_10_26).

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Arquitectura general](01-arquitectura-general.md)
- [Especificación de hardware](02-especificacion-hardware.md)
- [Especificación de simulación](03-especificacion-simulacion.md)
- [Estado de avance](08-estado-avance.md)