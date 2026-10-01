# 🛴 SmartGyro Controller: BLE Protocol Analysis & Reverse Engineering

[🇪🇸 Español](#-español) | [🇬🇧 English](#-english)

---

<a name="-español"></a>
## 🇪🇸 Español

![Python](https://img.shields.io/badge/Python-3.x-blue.svg)
![Ghidra](https://img.shields.io/badge/Ghidra-Static_Analysis-red)
![Reverse Engineering](https://img.shields.io/badge/Reverse_Engineering-ARM_Cortex_M-lightgrey)
![Status](https://img.shields.io/badge/Status-Educational_Only-yellow)

### ⚠️ Disclaimer (Aviso Legal)
> **ESTE REPOSITORIO ES ESTRICTAMENTE EDUCATIVO Y DE AUDITORÍA DE SEGURIDAD.** 
> La modificación del firmware de un Vehículo de Movilidad Personal (VMP) para alterar sus límites de velocidad de fábrica incumple la normativa vigente (DGT en España) y puede comprometer gravemente la seguridad física del vehículo y de los peatones.
> 
> **NO** se proporcionan archivos `.bin` precompilados.
> **NO** se publican las direcciones de memoria exactas ni los payloads hexadecimales críticos.
> Este proyecto documenta vulnerabilidades lógicas en el ecosistema IoT del controlador (pantalla) TFM13 con el **único** fin de demostrar conceptos de ingeniería inversa y análisis de   > protocolos binarios.
>
> **Tu garantia expirará si decides modificar de alguna forma cualquier componente del patin.**
>
> **No me responsabilizo de patinetes quitados, brickeados,**
> **guerras termonucleares, o de que te despidan por llegar tarde por que tu patin ha decidido morir.**
> 
> **Usa este conocimiento bajo tu propia responsabilidad.**

---

### 📖 Abstract
Este proyecto documenta la ingeniería inversa realizada sobre un controlador de motor, ensamblados en patinetes eléctricos como el SmartGyro Crossover Dual Max 2. El análisis abarca desde la interceptación y emulación del tráfico Bluetooth Low Energy (BLE) mediante Python, hasta el análisis estático del firmware del microcontrolador ARM utilizando Ghidra.

### 🔬 1. BLE Protocol Analysis (Puente BLE-UART)

La comunicación entre el display o la app móvil y el controlador del motor se realiza a través de un módulo Bluetooth que actúa como puente UART de comunicación serial. 

#### GATT Characteristics
- **TX/Notify:** `0000ffe2-0000-1000-8000-00805f9b34fb` (El controlador envía telemetría).
- **RX/Write:** `0000ffe1-0000-1000-8000-00805f9b34fb` (La app envía comandos).

#### Estructura de la Trama (Payload)
Los paquetes de datos utilizan una cabecera específica, comandos, subcomandos, un payload de longitud variable y un **CRC16** final para garantizar la integridad de los datos.
- **`5A 21` (`!`):** Comandos de escritura/lectura de parámetros de estado.
- **`5A 23` (`#`):** Comandos de acción en tiempo real (modos de conducción, acelerador).
- **`5A B1`:** Telemetría de vuelta (estado de la batería, velocidad mostrada en el display).

Se ha desarrollado un script en Python utilizando la librería `bleak` para interactuar con estas características, monitorizar el tráfico y emular el *handshake* inicial (autenticación) que exige el dispositivo para aceptar comandos.

### 🛠 2. Static Analysis & Reverse Engineering (Ghidra)

Se extrajo el firmware original del controlador (`.bin`) y se desensambló la arquitectura ARM Cortex-M utilizando Ghidra. El objetivo fue trazar cómo el microcontrolador procesa el buffer de entrada inyectado por el puente BLE-UART y experimentar con las direcciones de memoria.

#### Mapeo de Funciones Clave
1. **Parser BLE:** Se identificó la rutina encargada de leer el array de bytes entrante y derivar el flujo de ejecución según la cabecera recibida (`0x21` o `0x23`).
2. **Hard Limits por Software:** Se localizó la subrutina que evalúa el perfil de conducción actual. El firmware original aplica restricciones estrictas comprobando un *flag* en RAM. Si el flag está a `0`, se fuerza un salto condicional (`cmp`) hacia un tope lógico que corta la velocidad física del motor a los límites legales (aprox. 25 km/h).

#### PoC de Vulnerabilidad: Intercepción Lógica (Binary Patching)
Durante el análisis, se descubrió que es posible aplicar parches a nivel de ensamblador sobre funciones secundarias inofensivas.
Por ejemplo, interceptando la lógica del comando **"Lock" (Candado virtual del vehículo)**, es posible modificar las instrucciones ARM para que escriban un `1` o un `0` directamente en la dirección de memoria (`0x2000XXXX`) que gobierna la limitación de velocidad. 

Esto anula la función original de bloqueo de motor, pero la convierte en un interruptor de rendimiento no autenticado, manteniendo el límite estricto de fábrica por defecto en cada reinicio del vehículo por la volatilidad de la RAM, lo que lo hace ideal para su uso en circuito cerrado.

### 🚀 Próximos pasos
- [x] Sniffing del protocolo BLE y *handshake* de autenticación.
- [x] Ingeniería inversa de la aplicación móvil.
- [x] Ingeniería inversa del `.bin` y mapeo en Ghidra.
- [x] Desarrollo de script de flasheo en Python (`bleak`) con cálculo de CRC16 y CRC32 para que la pantalla acepte la actualización.
- [ ] (Futuro) Desarrollo de una interfaz de telemetría personalizada para monitorizar el estado en bruto del hardware (voltajes, temperaturas, logs) en SwiftUI.

**Autor:** Iker Perea  
**Tecnologías y Herramientas:** Ghidra, ARM Assembly, Python, `bleak`, Hex Editors, Ingeniería Inversa de Hardware/IoT, Bluetooth.

---

<br>

<a name="-english"></a>
## 🇬🇧 English

![Python](https://img.shields.io/badge/Python-3.x-blue.svg)
![Ghidra](https://img.shields.io/badge/Ghidra-Static_Analysis-red)
![Reverse Engineering](https://img.shields.io/badge/Reverse_Engineering-ARM_Cortex_M-lightgrey)
![Status](https://img.shields.io/badge/Status-Educational_Only-yellow)

### ⚠️ Disclaimer (Legal Notice)
> **THIS REPOSITORY IS STRICTLY FOR EDUCATIONAL AND SECURITY AUDITING PURPOSES.** 
> Modifying the firmware of a Personal Mobility Vehicle (PMV) to alter factory speed limits violates current regulations and may severely compromise the physical safety of the vehicle and pedestrians.
> 
> **NO** precompiled `.bin` files are provided.
> **NO** exact memory addresses or critical hexadecimal payloads are published.
> This project documents logical vulnerabilities within the IoT ecosystem of the TFM13 controller (display) **solely** for the purpose of demonstrating reverse engineering and binary
> protocol analysis concepts.
>
> **Your warranty will void if you modify in anyway your escooter.**
>
> **I am not responsible for bricked devices, taken escooters**
> **thermonuclear war, or you getting fired because the escooter decided to die.**
>
> **Use this knowledge under your self responsability.**

---

### 📖 Abstract
This project documents the reverse engineering performed on a motor controller assembled in electric scooters such as the SmartGyro Crossover Dual Max 2. The analysis ranges from capturing and emulating Bluetooth Low Energy (BLE) traffic using Python to static analysis of the ARM microcontroller firmware using Ghidra.

### 🔬 1. BLE Protocol Analysis (BLE-UART Bridge)

Communication between the display or mobile app and the motor controller is handled via a Bluetooth module acting as a serial UART bridge. 

#### GATT Characteristics
- **TX/Notify:** `0000ffe2-0000-1000-8000-00805f9b34fb` (The controller sends telemetry).
- **RX/Write:** `0000ffe1-0000-1000-8000-00805f9b34fb` (The app sends commands).

#### Frame Structure (Payload)
Data packets use a specific header, commands, subcommands, a variable-length payload, and a final **CRC16** to ensure data integrity.
- **`5A 21` (`!`):** State parameter read/write commands.
- **`5A 23` (`#`):** Real-time action commands (riding modes, throttle).
- **`5A B1`:** Return telemetry (battery status, speed shown on the display).

A Python script has been developed using the `bleak` library to interact with these characteristics, monitor traffic, and emulate the initial handshake (authentication) required by the device to accept commands.

### 🛠 2. Static Analysis & Reverse Engineering (Ghidra)

The original controller firmware (`.bin`) was extracted, and the ARM Cortex-M architecture was disassembled using Ghidra. The goal was to trace how the microcontroller processes the input buffer injected by the BLE-UART bridge and experiment with memory addresses.

#### Key Functions Mapping
1. **BLE Parser:** Identified the routine responsible for reading the incoming byte array and branching the execution flow based on the received header (`0x21` or `0x23`).
2. **Software Hard Limits:** Located the subroutine that evaluates the current driving profile. The original firmware enforces strict restrictions by checking a flag in RAM. If the flag is `0`, a conditional branch (`cmp`) forces a jump to a logical cap that cuts the physical speed of the motor down to legal limits (approx. 25 km/h).

#### Vulnerability PoC: Logical Interception (Binary Patching)
During the analysis, it was discovered that assembly-level patches can be applied to secondary, harmless functions.
For example, by intercepting the logic of the **"Lock" (Virtual Vehicle Immobilizer)** command, it is possible to modify the ARM instructions to write a `1` or a `0` directly into the memory address (`0x2000XXXX`) that governs the speed limitation. 

This disables the original motor locking function, turning it into an unauthenticated performance switch while keeping the strict factory limit active by default upon every vehicle reboot due to RAM volatility, making it ideal for closed-circuit use.

### 🚀 Next Steps
- [x] BLE protocol sniffing and authentication handshake.
- [x] Mobile application reverse engineering.
- [x] Firmware (`.bin`) reverse engineering and Ghidra mapping.
- [x] Development of a Python flashing script (`bleak`) with CRC16 and CRC32 calculation so that the display accepts the update.
- [ ] (Future) Development of a custom telemetry interface to monitor raw hardware status (voltages, temperatures, logs) in SwiftUI.

**Author:** Iker Perea  
**Technologies and Tools:** Ghidra, ARM Assembly, Python, `bleak`, Hex Editors, IoT/Hardware Reverse Engineering, Bluetooth.
