# 🛴 SmartGyro Controller: BLE Protocol Analysis & Reverse Engineering

![Python](https://img.shields.io/badge/Python-3.x-blue.svg)
![Ghidra](https://img.shields.io/badge/Ghidra-Static_Analysis-red)
![Reverse Engineering](https://img.shields.io/badge/Reverse_Engineering-ARM_Cortex_M-lightgrey)
![Status](https://img.shields.io/badge/Status-Educational_Only-yellow)

## ⚠️ Disclaimer (Aviso Legal)
> **ESTE REPOSITORIO ES ESTRICTAMENTE EDUCATIVO Y DE AUDITORÍA DE SEGURIDAD.** 
> La modificación del firmware de un Vehículo de Movilidad Personal (VMP) para alterar sus límites de velocidad de fábrica incumple la normativa vigente (DGT en España) y puede comprometer gravemente la seguridad física del vehículo y de los peatones.
> 
> **NO** se proporcionan archivos `.bin` precompilados.
> **NO** se publican las direcciones de memoria exactas ni los payloads hexadecimales críticos.
> Este proyecto documenta vulnerabilidades lógicas en el ecosistema IoT del controlador (pantalla) TFM13 con el único fin de demostrar conceptos de ingeniería inversa y análisis de protocolos binarios. 

---

## 📖 Abstract
Este proyecto documenta la ingeniería inversa realizada sobre un controlador de motor, ensamblados en patinetes eléctricos como el SmartGyro Crossover Dual Max 2. El análisis abarca desde la interceptación y emulación del tráfico Bluetooth Low Energy (BLE) mediante Python, hasta el análisis estático del firmware del microcontrolador ARM utilizando Ghidra.

## 🔬 1. BLE Protocol Analysis (Puente BLE-UART)

La comunicación entre el display o la app móvil y el controlador del motor se realiza a través de un módulo Bluetooth que actúa como puente UART de comunicación serial. 

### GATT Characteristics
- **TX/Notify:** `0000ffe2-0000-1000-8000-00805f9b34fb` (El controlador envía telemetría).
- **RX/Write:** `0000ffe1-0000-1000-8000-00805f9b34fb` (La app envía comandos).

### Estructura de la Trama (Payload)
Los paquetes de datos utilizan una cabecera específica, comandos, subcomandos, un payload de longitud variable y un **CRC16** final para garantizar la integridad de los datos.
- **`5A 21` (`!`):** Comandos de escritura/lectura de parámetros de estado.
- **`5A 23` (`#`):** Comandos de acción en tiempo real (modos de conducción, acelerador).
- **`5A B1`:** Telemetría de vuelta (estado de la batería, velocidad mostrada en el display).

Se ha desarrollado un script en Python utilizando la librería `bleak` para interactuar con estas características, monitorizar el tráfico y emular el *handshake* inicial (autenticación) que exige el dispositivo para aceptar comandos.

## 🛠 2. Static Analysis & Reverse Engineering (Ghidra)

Se extrajo el firmware original del controlador (`.bin`) y se desensambló la arquitectura ARM Cortex-M utilizando Ghidra. El objetivo fue trazar cómo el microcontrolador procesa el buffer de entrada inyectado por el puente BLE-UART y experimentar con las direcciones de memoria.

### Mapeo de Funciones Clave
1. **Parser BLE:** Se identificó la rutina encargada de leer el array de bytes entrante y derivar el flujo de ejecución según la cabecera recibida (`0x21` o `0x23`).
2. **Hard Limits por Software:** Se localizó la subrutina que evalúa el perfil de conducción actual. El firmware original aplica restricciones estrictas comprobando un *flag* en RAM. Si el flag está a `0`, se fuerza un salto condicional (`cmp`) hacia un tope lógico que corta la velocidad física del motor a los límites legales (aprox. 25 km/h).

### PoC de Vulnerabilidad: Intercepción Lógica (Binary Patching)
Durante el análisis, se descubrió que es posible aplicar parches a nivel de ensamblador sobre funciones secundarias inofensivas.
Por ejemplo, interceptando la lógica del comando **"Lock" (Candado virtual del vehículo)**, es posible modificar las instrucciones ARM para que escriban un `1` o un `0` directamente en la dirección de memoria (`0x2000XXXX`) que gobierna la limitación de velocidad. 

Esto anula la función original de bloqueo de motor, pero la convierte en un interruptor de rendimiento no autenticado, manteniendo el límite estricto de fábrica por defecto en cada reinicio del vehículo por la volatilidad de la RAM, lo que lo hace ideal para su uso en circuito cerrado.

## 🚀 Próximos pasos
- [x] Sniffing del protocolo BLE y *handshake* de autenticación.
- [x] Ingeniería inversa de la aplicación móvil.
- [x] Ingeniería inversa del `.bin` y mapeo en Ghidra.
- [x] Desarrollo de script de flasheo en Python (`bleak`) con cálculo de CRC16 y CRC32 para que la pantalla acepte la actualización.
- [ ] (Futuro) Desarrollo de una interfaz de telemetría personalizada para monitorizar el estado en bruto del hardware (voltajes, temperaturas, logs) en SwiftUI.

**Autor:** Iker Perea  
**Tecnologías y Herramientas:** Ghidra, ARM Assembly, Python, `bleak`, Hex Editors, Ingeniería Inversa de Hardware/IoT, Bluetooth.
