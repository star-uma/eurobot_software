# 07 - Guía de onboarding para nuevos integrantes

## 1. Instalación completa (ordenador de desarrollo)

Desde la carpeta del meta-repositorio `eurobot_software`:

```bash
# 1. Clonar el repositorio
git clone https://github.com/star-uma/eurobot_software.git
cd eurobot_software

# 2. Ejecutar el instalador automático
./instalador.sh
```

El instalador:
1. Crea el workspace en `eurobot_ws`.
2. Instala `python3-vcstool`.
3. Importa los sub-repositorios definidos en `robot.repos` (comun, real, simulacion, firmware_esp32).
4. Instala dependencias ROS 2 con `rosdep`.
5. Compila con `colcon build --symlink-install`.

Después de instalar, en cada terminal nueva:
```bash
source eurobot_ws/install/setup.bash
```

## 2. Instalación solo robot

```bash
git clone https://github.com/star-uma/eurobot_software.git
cd eurobot_software
./instalador_robot.sh
```

## 3. Actualizar repositorios

```bash
# Repositorio principal
git pull origin main

# Sub-repositorios
cd eurobot_ws
vcs pull eurobot_ws/src .
```

## 4. Lanzar simulación

```bash
source eurobot_ws/install/setup.bash
ros2 launch eurobot_software_simulacion sim.launch.py
```

En otra terminal para teleop por teclado:
```bash
source eurobot_ws/install/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

## 5. Ejecutar nodos de visión

```bash
source eurobot_ws/install/setup.bash
ros2 run eurobot_software_vision vision_node
ros2 run eurobot_software_vision filtro_lidar  # si se añade como entrypoint
```

## 6. Cómo añadir un nodo nuevo (flujo SDD)

1. **Escribir la especificación SDD** en `docs/` usando `plantilla-especificacion.md`.
2. **Crear el paquete** en el sub-repo correspondiente (normalmente `eurobot_software_comun`).
3. **Añadir el nodo** al `setup.py` / `CMakeLists.txt` y registrar el entrypoint.
4. **Actualizar launch files** si el nodo debe lanzarse automáticamente.
5. **Compilar:** `colcon build --packages-select <nombre_paquete> --symlink-install`.
6. **Actualizar `docs/08-estado-avance.md`** con lo implementado.

## 7. Comandos útiles

```bash
# Ver nodos activos
ros2 node list

# Ver topics activos
ros2 topic list

# Ver información de un topic
ros2 topic info /nombre_topic

# Ver parámetros de un nodo
ros2 param list
ros2 param get <nodo> <parametro>

# Visualizar TF en RViz2
ros2 run rviz2 rviz2

# Ver logs de un nodo
ros2 run <paquete> <nodo> --ros-args --log-level debug
```

## 8. Repositorios del proyecto

| Repo | URL |
|------|-----|
| eurobot_software (meta) | https://github.com/star-uma/eurobot_software.git |
| eurobot_software_comun | https://github.com/star-uma/eurobot_software_comun.git |
| eurobot_software_real | https://github.com/star-uma/eurobot_software_real.git |
| eurobot_software_simulacion | https://github.com/star-uma/eurobot_software_simulacion.git |
| eurobot_software_firmware_esp32 | https://github.com/star-uma/eurobot_software_firmware_esp32.git |

## 9. Normas del equipo

- Toda feature nueva empieza por una especificación SDD en `docs/`.
- El README se mantiene breve (solo descripción e instalación).
- Los parámetros físicos solo se tocan en `parametros_robot.yaml`.
- La documentación se actualiza junto al código (no después).

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Plantilla de especificación](plantilla-especificacion.md)
- [Estado de avance](08-estado-avance.md)
- [Arquitectura general](01-arquitectura-general.md)