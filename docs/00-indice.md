# Índice de documentación SDD - Eurobot 2027

## Especificaciones generales (estado actual)
- [01 - Arquitectura general](01-arquitectura-general.md) → Mapa de paquetes, nodos, topics, launch files
- [02 - Especificación de hardware](02-especificacion-hardware.md) → URDF, parámetros físicos, límites del robot
- [03 - Especificación de simulación](03-especificacion-simulacion.md) → Gazebo Ignition, mundos, bridges, lanzamiento
- [04 - Especificación de robot real](04-especificacion-robot-real.md) → Firmware ESP32, drivers, diferencias sim→real
- [05 - Especificación de visión](05-especificacion-vision.md) → vision_node, ArUco, filtro lidar
- [06 - Especificación de ros2_control](06-especificacion-ros2-control.md) → diff_cont, actuator_cont, controllers.yaml

## Especificaciones de features
- [09 - Control Xbox](09-especificacion-xbox.md) → Control remoto por mando Xbox (sim + real) — especificación escrita, paquete creado
- [10 - Actuador real](10-especificacion-actuador-real.md) → Rediseño del actuador: pilares laterales, barra rotativa, ventosas — pendiente

## Guías
- [07 - Guía de onboarding](07-guia-onboarding-companeros.md) → Cómo clonar, instalar, compilar, lanzar, añadir un nodo
- [Plantilla de especificación](plantilla-especificacion.md) → Plantilla vacía para nuevas features (guía para la IA)

## Estado del proyecto
- [08 - Estado de avance](08-estado-avance.md) → Qué está hecho, qué falta, bugs conocidos (actualizar tras cada prioridad)

---

## Prioridades a corto plazo
1. Documentación SDD (esta fase)
2. Control Xbox
3. Actuador real
4. Nav2 + odometría
5. Detección ArUco para centrado de caja
6. Sensores rivales/objetos (inferiores + lidar superior)
7. FSM de competición final
8. Perfeccionamiento iterativo