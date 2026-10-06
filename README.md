# 🐜 Enjambre de 3 Carritos con Algoritmo de Colonia de Hormigas (ACO) en ESP32 y Gemelo Digital en PyBullet

##  Descripción del problema

Tres carritos físicos (basados en nodos **ESP32**) deben encontrar la ruta óptima desde un punto **A** hasta una meta en un laberinto/almacén. El algoritmo de optimización por colonia de hormigas (**ACO**) corre dentro de cada ESP32, las feromonas se intercambian por red (ESP-NOW / modo AP), y el resultado se refleja en una simulación **PyBullet** donde 3 robots virtuales replican el comportamiento. Todo el entorno virtual se ejecuta dentro de un contenedor **Docker**.

---

##  Solución: Algoritmo ACO (Ant Colony Optimization)

Cada "hormiga" (carrito) explora la grilla del laberinto dejando trazas de feromona en las celdas visitadas. La probabilidad de elegir el siguiente movimiento depende de:

**P(i) = τ(i)^α · η(i)^β / Σ τ(j)^α · η(j)^β**

donde:
- `τ(i)` = nivel de feromona del camino *i*
- `η(i)` = heurística (inverso de la distancia a la meta)
- `α` = peso de la feromona
- `β` = peso de la heurística

### Ciclo del algoritmo
1. Cada carrito genera una ruta desde **A** hasta la meta (o se detiene al agotar pasos).
2. **Evaporación**: `τ ← τ·(1 − ρ)` (ρ = 0.15).
3. **Refuerzo**: las rutas exitosas depositan feromona `Q / longitud_ruta`.
4. Iterar hasta converger → la ruta más corta acumula mayor feromona y emerge como **ruta óptima**.

### Pseudocódigo
```
PARA cada iteración:
    PARA cada hormiga k:
        ruta_k = explorar(inicio → meta)  // elección probabilística P(i)
    τ = τ·(1-ρ)                            // evaporación
    PARA cada ruta exitosa:
        τ[celda] += Q / longitud(ruta)     // depósito
    actualizar mejor_ruta
```

---

##  Estructura de carpetas del proyecto

```
ACO_ESP32_PyBullet/
├── README.md                        # Este documento
├── simulacion/
│   └── index.html                   # Simulación ACO del laberinto (botón de inicio)
├── esp32/
│   └── aco_esp32.ino                # Firmware ACO + ESP-NOW (mundo físico)
├── pybullet/
│   └── gemelo_digital.py            # Gemelo digital con 3 robots virtuales
└── docker/
    └── Dockerfile                   # Contenedor con PyBullet
```

---

##  Código respectivo

### 1) Firmware ESP32 (`esp32/aco_esp32.ino`)
Cada ESP32 ejecuta ACO y difunde su matriz de feromonas por ESP-NOW:

```cpp
void pasoACO() {
  for (int h = 0; h < NUM_HORMIGAS; h++) {
    // Exploración probabilística guiada por feromona + heurística
    ...
  }
  for (int r = 0; r < FILAS; r++)
    for (int c = 0; c < COLS; c++)
      tau[r][c] = tau[r][c]*(1-RHO) + mejor[r*COLS+c];
}

void loop() {
  pasoACO();
  enviarFeromonas();   // comparación con el resto del enjambre
  delay(500);
}
```

### 2) Gemelo digital PyBullet (`pybullet/gemelo_digital.py`)
```python
mejor = correr_aco()                       # 40 iteraciones, 8 hormigas
for i in range(3):                         # enjambre de 3 carritos virtuales
    robots.append(p.loadURDF("r2d2.urdf", ...))
for paso in mejor:                         # los 3 robots replican la ruta óptima
    ...
```

### 3) Parte de red
- **Modo AP / ESP-NOW**: cada ESP32 envía su matriz `tau` y recibe la de sus vecinos; se promedia para fusionar feromonas (`tau = (tau+remoto)/2`).

### 4) Simulación web (`simulacion/index.html`)
Archivo HTML autocontenido: genera un laberinto aleatorio, ejecuta ACO con 15 hormigas y un botón **"▶ Iniciar simulación"** muestra la exploración (rutas rojas), el campo de feromonas (verde) y la ruta óptima final recorrida por 🚗.
<img width="1097" height="893" alt="image" src="https://github.com/user-attachments/assets/cd220b5e-c202-4e28-8d54-f1b9e8af0d31" />

---

## ▶ Cómo ejecutar

**Simulación web (recomendado para evidencias):**
- Abrir `simulacion/index.html` en cualquier navegador y presionar **Iniciar simulación**.
- [file:///C:/Users/Camilo/Desktop/index.html] 

**Gemelo digital con Docker:**
```bash
cd docker
docker build -t gemelo-aco .
docker run --rm -it gemelo-aco
```

**Firmware ESP32:**
- Abrir `esp32/aco_esp32.ino` en Arduino IDE / PlatformIO con placa ESP32 seleccionada y cargarlo en los 3 nodos.

---

##  Resultados esperados

1. Los 3 ESP32 convergen en la misma distribución de feromonas → ruta única óptima.
2. El gemelo digital muestra 3 robots recorriendo la misma trayectoria.
3. La simulación HTML evidencia visualmente convergencia del enjambre, refuerzo de feromonas y ruta mínima A→🏁.

<img width="1593" height="178" alt="image" src="https://github.com/user-attachments/assets/e4912ebd-5d7e-4e8e-a0e4-de4b9f7637e5" />
- Resultados en la terminal de VS Code.

<img width="1023" height="785" alt="image" src="https://github.com/user-attachments/assets/7ba2ff31-2f79-49f5-a486-8700ba011c54" />
- Laberinto en PyBullet, cuadro azul el inicio o punto A, cuadro rojo es la meta.

- Video (link)
---

##  Referencias
- Dorigo, M., & Stützle, T. *Ant Colony Optimization*. MIT Press, 2004.
- Documentación ESP-NOW y PyBullet.
