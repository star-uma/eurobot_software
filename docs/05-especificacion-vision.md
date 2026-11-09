# 05 - Especificación de visión artificial

## 1. Paquete: eurobot_software_vision

Contiene dos nodos Python basados en rclpy y OpenCV.

## 2. Nodo: vision_node

Archivo: `eurobot_software_vision/vision_node.py`

**Función:** Detecta marcadores ArUco en la imagen de la cámara y los visualiza.

**Interfaces:**
- Suscribe: `/camera/image_raw` (`sensor_msgs/Image`)
- Publica: (ningún topic ROS; solo visualización local con `cv2.imshow`)
- Nodo llamado: `vision_node`

**Detalle de implementación:**
- Diccionario ArUco: `cv2.aruco.DICT_4X4_50`.
- Parámetros: `cv2.aruco.DetectorParameters_create()`.
- Función de detección: `cv2.aruco.detectMarkers()` (API OpenCV 4.5, compatible ROS 2 Humble).
- Dibuja marcadores detectados con `cv2.aruco.drawDetectedMarkers()`.
- Loggea los IDs detectados: `"Cajas detectadas (IDs): {ids.flatten()}"`.
- Ventana de depuración: `"Eurobot - Vision Debug"`, `cv2.waitKey(1)`.

**Entrypoint (setup.py):**
```
vision_node = eurobot_software_vision.vision_node:main
```

## 3. Nodo: filtro_lidar

Archivo: `eurobot_software_vision/filtro_lidar.py`

**Función:** Filtra el escaneo lidar eliminando rangos mayores a una distancia máxima.

**Interfaces:**
- Suscribe: `/scan` (`sensor_msgs/LaserScan`)
- Publica: `/scan_filtered` (`sensor_msgs/LaserScan`)
- Nodo llamado: `lidar_filter_node`

**Detalle de implementación:**
- `distancia_maxima = 1.0` m.
- Rangos > 1.0 m se convierten en `float('inf')`.
- `range_max` del mensaje filtrado se establece a `distancia_maxima`.
- Publica el mensaje filtrado.

## 4. Cámara en Gazebo

Definida en `urdf/camara.xacro`:
- Posición: `xyz=0.305 0 0.15` respecto a `chassis`.
- Joint óptico: rotación `-pi/2, 0, -pi/2` para corregir coordenadas OpenCV (Z adelante, X derecha).
- Sensor Gazebo: `camera_sensor`, topic `camera/image_raw`, 640x480, R8G8B8, FOV horizontal 1.089, clip 0.02–8.0 m, 30 Hz.

## 5. Lidar en Gazebo

Definida en `urdf/lidar.xacro`:
- Posición: `xyz=0.1 0 0.25` respecto a `chassis`.
- Sensor Gazebo: `lidar`, tipo `gpu_lidar`, topic `scan`, 360 samples, rango 0.12–8.0 m, resolución 0.01 m, 10 Hz.

## 6. Pendientes / conocidos

- `vision_node` solo muestra resultados en ventana local (`imshow`); no publica topic con las posiciones de los ArUco. Para usarlos en centrado de caja (prioridad 5) habrá que publicar un topic con ID y pose estimada.
- No hay nodo de fusión cámara + lidar + odometría todavía.
- El filtro lidar es estático (1.0 m fijo); podría parametrizarse.

---

## Documentación relacionada

- [Índice general](00-indice.md)
- [Arquitectura general](01-arquitectura-general.md)
- [Especificación de simulación](03-especificacion-simulacion.md)
- [Estado de avance](08-estado-avance.md)