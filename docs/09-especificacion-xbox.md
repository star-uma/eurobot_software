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

## 3. Interfaces ROS 2

### Topics publicados
| Topic | Tipo | Frecuencia | Descripción |
|-------|------|------------|-------------|
| `/cmd_vel` | `geometry_msgs/Twist` | Según mensajes `/joy` | Comando de velocidad para `diff_cont` |

### Topics suscritos
| Topic | Tipo | Descripción |
|-------|------|-------------|
| `/joy` | `sensor_msgs/Joy` | Entrada del mando Xbox (producido por el nodo `joy` de ROS 2) |

### Parámetros
| Parámetro | Tipo | Valor por defecto | Descripción |
|-----------|------|-------------------|-------------|
| `require_enable_button` | bool | `true` | Exigir botón de habilitación |
| `enable_button` | int | `7` | Índice del botón habilitador (RB) |
| `axis_linear_x` | int | `1` | Índice del eje para velocidad lineal |
| `scale_linear_x` | double | `0.5` | Escala de velocidad lineal (m/s) |
| `axis_angular_yaw` | int | `0` | Índice del eje para velocidad angular |
| `scale_angular_yaw` | double | `1.0` | Escala de velocidad angular (rad/s) |

## 4. Diseño
- **Flujo de datos:** Mando Xbox → nodo `joy` (driver ROS 2) → topic `/joy` → `xbox_teleop_node` → topic `/cmd_vel` → `diff_cont` → ruedas del robot.
- **Lógica de decisión:** Si `require_enable_button` es true y el botón RB no está presionado, se publica velocidad cero. De lo contrario, se mapean los ejes del joystick a `Twist.linear.x` y `Twist.angular.z` aplicando los factores de escala.
- **Sim-to-real:** El mismo nodo se ejecuta en ambos entornos. En simulación se lanza junto a `sim.launch.py`; en el robot real se ejecuta con el driver del mando Xbox configurado.

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
  - `docs/01-arquitectura-general.md` → mapa de paquetes, topics, nodos, launch files actualizados
  - `docs/00-indice.md` → referencia a especificación Xbox
  - `docs/08-estado-avance.md` → prioridad 2 actualizada
- **NO tocar:**
  - `sim.launch.py` (no se modifica; el paquete xbox se lanza de forma independiente)
  - `controllers.yaml` (los parámetros de velocidad máxima ya están definidos ahí)

## 6. Criterios de aceptación
- [ ] El nodo `xbox_teleop_node` compila y se ejecuta sin errores.
- [ ] Al presionar RB y mover el joystick izquierdo, el robot responde en `/cmd_vel`.
- [ ] Sin presionar RB, el robot no se mueve (velocidad cero).
- [ ] Funciona en simulación con `diff_cont` y en robot real (cuando esté integrado).

## 7. Notas para la IA
- **Convenciones de nombres:** topics en snake_case, nodo en `xbox_teleop_node`, paquete en `eurobot_software_xbox`.
- **Versiones:** rclpy, `sensor_msgs/Joy`, `geometry_msgs/Twist` (versión de ROS 2 según entorno: Humble en desarrollo, Jazzy en robot real).
- **Restricciones:** No modificar `sim.launch.py` ni `controllers.yaml`. El mapeo de ejes y botones coincide con `config/joy_teleop.yaml`.
- **Dependencia:** Requiere el nodo `joy` de ROS 2 para publicar en `/joy` desde el mando físico.