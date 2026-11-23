# 09 - Control remoto por mando Xbox (eurobot_software_xbox)

## 1. Contexto y objetivo
- **Problema que resuelve:** Permitir controlar el movimiento del robot mediante un mando Xbox para facilitar pruebas de movimiento y demos, sin depender exclusivamente del teclado (`teleop_twist_keyboard`) ni de código autónomo.
- **Entorno de ejecución:** ☐ Simulación ☐ Robot real ☒ Ambos
- **Nodos existentes afectados:** `diff_cont` (recibe `/cmd_vel`).
- **Nodos nuevos que crea:** `xbox_teleop_node` (paquete `eurobot_software_xbox`).

## 2. Requisitos funcionales
| ID | Requisito | Prioridad |
|----|-----------|-----------|
| RF-001 | Leer mensajes del topic `/joy` (`sensor_msgs/Joy`) y publicar `/cmd_vel` (`geometry_msgs/Twist`). | Alta |
| RF-002 | Botón RB (índice 7) como habilitador de seguridad: si no está presionado, el robot no se mueve. | Alta |
| RF-003 | Eje Y del joystick izquierdo → velocidad lineal (máx 0.5 m/s). | Alta |
| RF-004 | Eje X del joystick izquierdo → velocidad angular (máx 1.0 rad/s). | Alta |
| RF-005 | Funcionar tanto en simulación (Gazebo) como en robot real sin cambios de código. | Alta |
| RF-006 | Solo para pruebas y demos, no para competición oficial. | Media |
| RF-007 | Eje 7 (flecha arriba/abajo) mueve el `elevator_joint` del actuador entre 0.0 y 0.38 m, **sin necesidad de RB**. | Alta |
| RF-008 | Botón A para activar la ventosa: **pendiente**, requiere mecanismo de agarre (plugin Gazebo) y, en el robot real, electroválvula. | Baja |

## 3. Interfaces ROS 2

### Topics publicados
| Topic | Tipo | Frecuencia | Descripción |
|-------|------|------------|-------------|
| `/cmd_vel` | `geometry_msgs/Twist` | Según mensajes `/joy` | Comando de velocidad para `diff_cont` |
| `/actuator_cont/joint_trajectory` | `trajectory_msgs/JointTrajectory` | 20 Hz (solo al cambiar el objetivo) | Posición objetivo del `elevator_joint` |

### Topics suscritos
| Topic | Tipo | Descripción |
|-------|------|-------------|
| `/joy` | `sensor_msgs/Joy` | Entrada del mando Xbox (producido por el nodo `joy` de ROS 2) |
| `/joint_states` | `sensor_msgs/JointState` | Posición actual del `elevator_joint` (para inicializar el objetivo sin saltos) |

### Parámetros
| Parámetro | Tipo | Valor por defecto | Descripción |
|-----------|------|-------------------|-------------|
| `require_enable_button` | bool | `true` | Exigir botón de habilitación |
| `enable_button` | int | `7` | Índice del botón habilitador (RB) |
| `axis_linear_x` | int | `1` | Índice del eje para velocidad lineal |
| `scale_linear_x` | double | `0.5` | Escala de velocidad lineal (m/s) |
| `axis_angular_yaw` | int | `0` | Índice del eje para velocidad angular |
| `scale_angular_yaw` | double | `1.0` | Escala de velocidad angular (rad/s) |
| `actuator_axis` | int | `7` | Índice del eje de la flecha (+1 arriba / -1 abajo) |
| `actuator_velocity` | double | `0.1` | Velocidad de desplazamiento del objetivo (m/s) |
| `actuator_min` | double | `0.0` | Límite inferior del actuador (m) |
| `actuator_max` | double | `0.38` | Límite superior del actuador (m) |
| `actuator_deadzone` | double | `0.5` | Umbral para considerar la flecha pulsada |
| `actuator_rate` | double | `20.0` | Frecuencia de publicación de la trayectoria (Hz) |

## 4. Diseño
- **Flujo de datos (movimiento):** Mando Xbox → `joy_node` → topic `/joy` → `xbox_teleop_node` → topic `/cmd_vel` → `diff_cont` → ruedas del robot.
- **Flujo de datos (actuador):** Mando Xbox → `/joy` → `xbox_teleop_node` (timer 20 Hz) → `/actuator_cont/joint_trajectory` → `actuator_cont` → `elevator_joint`.
- **Lógica de movimiento:** si `require_enable_button` es true y el botón RB no está presionado, se publica velocidad cero. Si no, se mapean los ejes del joystick a `Twist.linear.x` y `Twist.angular.z` aplicando los factores de escala.
- **Lógica del actuador:** la flecha (eje 7) integra el objetivo a `actuator_velocity` m/s, se limita a `[actuator_min, actuator_max]` y se publica una `JointTrajectory` solo cuando el objetivo cambia. **No requiere RB.** El objetivo se inicializa desde `/joint_states` para no dar saltos al arrancar.
- **Sello de tiempo:** la trayectoria se publica con `header.stamp = 0`, que el `JointTrajectoryController` interpreta como "empieza ahora". Esto evita el desajuste entre el reloj de pared del nodo y el reloj de simulación (`/clock`).
- **Sim-to-real:** el mismo nodo se ejecuta en ambos entornos. `xbox.launch.py` lanza `joy_node` + `xbox_teleop_node` y acepta `use_sim_time` (`true` en simulación, `false` en el robot real).

## 5. Archivos afectados
- **Creados:**
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/package.xml`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/setup.py`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/setup.cfg`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/resource/eurobot_software_xbox`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/eurobot_software_xbox/__init__.py`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/eurobot_software_xbox/xbox_teleop.py`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/launch/xbox.launch.py`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/config/joy_teleop.yaml`
  - `docs/09-especificacion-xbox.md`
- **Modificados:**
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_description/config/joy_teleop.yaml` → eliminado (movido al nuevo paquete)
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/package.xml` → añadidas dependencias `trajectory_msgs` y `joy`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/launch/xbox.launch.py` → lanza `joy_node` + nodo y acepta `use_sim_time`
  - `eurobot_ws/src/eurobot_software_comun/eurobot_software_xbox/eurobot_software_xbox/xbox_teleop.py` → control del actuador + sello de trayectoria a cero
  - `docs/01-arquitectura-general.md` → mapa de paquetes, topics, nodos, launch files actualizados
  - `docs/00-indice.md` → referencia a especificación Xbox
  - `docs/08-estado-avance.md` → prioridad 2 actualizada
- **NO tocar:**
  - `sim.launch.py` (no se modifica; el paquete xbox se lanza de forma independiente)
  - `controllers.yaml` (los parámetros de velocidad máxima ya están definidos ahí)

## 6. Criterios de aceptación
- [x] El nodo `xbox_teleop_node` compila y se ejecuta sin errores.
- [x] Al presionar RB y mover el joystick izquierdo, el robot responde en `/cmd_vel`.
- [x] Sin presionar RB, el robot no se mueve (velocidad cero).
- [x] Flecha arriba sube el `elevator_joint` y flecha abajo lo baja (límites 0.0–0.38 m), sin necesidad de RB.
- [x] Funciona en simulación (probado con Gazebo + `diff_cont` + `actuator_cont`).
- [ ] Robot real (cuando esté integrado; lanzar con `use_sim_time:=false`).
- [ ] Botón A activa la ventosa (pendiente del mecanismo de agarre).

## 7. Notas para la IA
- **Convenciones de nombres:** topics en snake_case, nodo en `xbox_teleop_node`, paquete en `eurobot_software_xbox`.
- **Versiones:** rclpy, `sensor_msgs/Joy`, `geometry_msgs/Twist` (versión de ROS 2 según entorno: Humble en desarrollo, Jazzy en robot real).
- **Restricciones:** No modificar `sim.launch.py` ni `controllers.yaml`. El mapeo de ejes y botones coincide con `config/joy_teleop.yaml`.
- **Sello de tiempo:** la `JointTrajectory` se publica con `header.stamp = 0` ("empieza ahora"). No volver a usar `get_clock().now()`: rompería el control en simulación por desajuste de relojes.
- **Reloj:** `xbox.launch.py` acepta `use_sim_time` (default `true` para simulación; `false` en el robot real).
- **Ventosa:** `suction_cup_link` es un link fijo sin plugin de agarre; el botón A queda pendiente de mecanismo en Gazebo y de electroválvula en el real.
- **Dependencia:** Requiere el paquete `joy` de ROS 2 (`ros-humble-joy`). `xbox.launch.py` ya lo lanza (`joy_node`); si el sistema usa `joy_linux`, cambiar a `joy_linux_node`.