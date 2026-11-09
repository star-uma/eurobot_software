# 02 - Especificación de hardware y parámetros del robot

## 1. Fuente única de verdad

Todos los parámetros físicos del robot se definen en:
```
eurobot_ws/src/eurobot_software_comun/eurobot_software_description/config/parametros_robot.yaml
```

Este archivo es leído tanto por el URDF (vía `xacro.load_yaml`) como por `sim.launch.py` (que fusiona sus valores en `controllers.yaml`). No debe haber parámetros físicos duplicados en otros archivos.

## 2. Parámetros de la base (chasis y tracción)

Sección `base`:

| Parámetro | Valor | Unidad | Descripción |
|-----------|-------|--------|-------------|
| wheel_radius | 0.0325 | m | Radio de rueda motriz |
| wheel_separation | 0.4 | m | Separación entre centros de ruedas |
| wheel_length | 0.04 | m | Longitud/ancho de la rueda |
| wheel_vel_max | 10 | — | Velocidad máxima rueda (escala) |
| chassis_length | 0.3 | m | Longitud del chasis (eje X) |
| chassis_width | 0.3 | m | Ancho del chasis (eje Y) |
| chassis_height | 0.15 | m | Altura del chasis (eje Z) |

**Geometría del chasis:**
- `base_link` es el ancla origen (0,0,0), sin geometría.
- `chassis` está unido por joint fixed `chassis_joint` con origen `-0.05 0 0`.
- Caja visual y de colisión de `0.3 x 0.3 x 0.15` m, masa 5.0 kg (inercia de caja).
- Ruedas motrices: `left_wheel_joint` y `right_wheel_joint` (tipo continuous), offset Y `+0.2` y `-0.2`, radio 0.0325 m, longitud 0.04 m, masa 1.0 kg cada una.
- Rueda loca `caster_wheel`: esfera, joint fixed al chasis, rozamiento nulo en Gazebo (mu1=mu2=0.001).

## 3. Parámetros del actuador (versión actual de prueba)

Sección `actuator`:

| Parámetro | Valor | Unidad | Descripción |
|-----------|-------|--------|-------------|
| offset_x | 0.10 | m | Desplazamiento X torre respecto a base_link |
| offset_y | 0.0 | m | Desplazamiento Y |
| offset_z | 0.15 | m | Desplazamiento Z (altura base torre) |
| elevator_mass | 0.5 | kg | Masa del carro ascensor |
| elevator_max_z | 0.38 | m | Límite superior articulación prismática |
| elevator_min_z | 0.0 | m | Límite inferior articulación prismática |
| elevator_vel_max | 0.5 | m/s | Velocidad máxima ascensor |
| tower_height | 0.45 | m | Altura de la torre guía |
| tower_width | 0.03 | m | Ancho de la torre guía |
| suction_mass | 0.1 | kg | Masa del cabezal de ventosas |
| suction_offset_x | 0.08 | m | Offset X ventosa |
| suction_radius | 0.02 | m | Radio del cilindro de ventosa |

**Estructura actual (test de unión):**
- `tower_link`: caja fija unida a base_link, torre estática.
- `elevator_joint`: tipo prismático, eje Z, límites 0.0–0.38 m, effort 500.0, velocity 0.5.
- `elevator_carriage_link`: carro que se mueve en Z.
- `suction_head_joint`: fixed, cilindro de ventosa al frente del carro.
- Problema reportado (log 5_11_26): ganancia a 0.1, fuerza insuficiente para mover la articulación prismática con carga.

## 4. URDF principal

Archivo: `urdf/robot.urdf.xacro`

Argumentos:
- `use_sim` (default: true)
- `controllers_yaml` (default: ruta a config/controllers.yaml)

Incluye:
1. `robot_core.xacro` — chasis, ruedas, rueda loca.
2. `lidar.xacro` — sensor lidar.
3. `camara.xacro` — cámara.
4. `actuador.xacro` — actuador (versión prueba).
5. `ros2_control.xacro` — interfaces ros2_control.

## 5. Materiales y colores (RViz)

- `white`, `blue`, `grey`, `black` definidos en `robot_core.xacro`.
- Gazebo ignora estos colores; requiere etiquetas `<gazebo>` específicas.

## 6. Pendientes / conocidos

- El chasis usa caja primitiva; se espera sustituir por mesh STL del CAD real (<30 caras recomendado para colisión).
- La inercia del chasis está comentada; se usa la macro `inertial_box` con masa 5.0 kg.
- El actuador actual es una prueba de unión prismática; requiere rediseño mecánico (ver prioridad 3: pilares laterales + barra rotativa + ventosas).

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Arquitectura general](01-arquitectura-general.md)
- [Especificación de simulación](03-especificacion-simulacion.md)
- [Especificación de ros2_control](06-especificacion-ros2-control.md)
- [Estado de avance](08-estado-avance.md)