<img src="https://www.fi.uba.ar/images/logo-fiuba.png" alt="Logo FIUBA" width="45%">

# **Estación de Monitoreo de Calidad de Aire, Clima y Monóxido de Carbono (CO)**
## **Informe de Avance**

**Autor:** Pereira, Nicolás Manuel

**Padrones:** 105269

**Curso y Grupo:** 01B-04  

**Fecha:** 20/11/2026 (2do Cuatrimestre 2026)  

---

### **1. Referencias de Estado de Avance**

A continuación se detalla el estado de cumplimiento de cada requisito planteado en `Propuesta.md`, utilizando el siguiente código de colores:

| Estado | Significado |
| :---: | :--- |
| 🟢 | **Implementado y verificado** |
| 🟡 | **En proceso de implementación** |
| 🔴 | **No se implemetara** |

---

### **2. Estado de Avance por Requisito**

#### **2.1 Sensado Ambiental y de Gases (ADC / I2C)**

| Req ID | Descripción | Estado | Observaciones |
| :---: | :--- | :---: | :--- |
| 1.1 | Lectura periódica del sensor de calidad de aire/VOCs **MQ-135** mediante canal analógico (`ADC1_IN0`) con divisor resistivo ($5\text{ V} \rightarrow 3,3\text{ V}$). | 🟢 | Probado en banco con adquisición por DMA y compensación de tensión. |
| 1.2 | Lectura de temperatura, humedad relativa y presión atmosférica mediante el sensor digital **BME280** por bus. | 🟢 | Probado en conjunto con los sensores MQ; lectura de ID (`0x60`) y compensación funcionando. |
| 1.3 | Filtrado digital por media móvil sobre las muestras crudas del ADC para estabilizar las mediciones. | 🟡 | Restan ajustar el tamaño de ventana y la conversión final a unidades de ppm. |
| 1.4 | Lectura del sensor de monóxido de carbono **MQ-7** mediante segundo canal analógico (`ADC1_IN1`) en modo *Scan* con DMA. | 🟢 | Probado en banco junto con el MQ-135 mediante `HAL_ADC_ConvCpltCallback`. |

#### **2.2 Interfaz de Usuario, Indicadores y Alarmas**

| Req ID | Descripción | Estado | Observaciones |
| :---: | :--- | :---: | :--- |
| 2.1 | Semáforo de 3 LEDs (Verde, Amarillo, Rojo) indicador del nivel de calidad de aire. | 🟡 | Lógica definida; no se verifico todavia. |
| 2.2 | Buzzer piezoeléctrico con generación de patrones sonoros no bloqueantes mediante *SysTick*. | 🟡 | Componente adquirido, todavia no se empezo a implementar. |
| 2.3 | Tres pulsadores físicos (*Menú/Modo*, *Incrementar/Aceptar*, *Silenciar Alarma*) con antirrebote por software. | 🟡 | Componente adquirido, todavia no se empezo a implementar. |
| 2.4 | DIP Switch para habilitación/deshabilitación física de alertas sonoras y selección de modo de energía. | 🟡 | Componente adquirido, todavia no se empezo a implementar. |
| 2.5 | Pantalla LCD 16x2 / OLED $\text{I}^2\text{C}$ para visualización de métricas en tiempo real y navegación del menú. | 🟡 | Componente adquirido, todavia no se empezo a implementar. |
| 2.6 | Alarma crítica prioritaria ante detección de $\text{CO} > 50\text{ ppm}$ con bloqueo de silenciado manual. | 🟡 | Diseñada en el diagrama de estados; pendiente de validación integrada. |

#### **2.3 Almacenamiento Persistente y Menú Interactivo**

| Req ID | Descripción | Estado | Observaciones |
| :---: | :--- | :---: | :--- |
| 3.1 | Menú interactivo local navegable con los pulsadores en modo `SET_UP`. | 🟡 | Estructura de pantallas diseñada; pendiente vinculación con el display. |
| 3.2 | Almacenamiento de umbrales de alarma y parámetros de calibración en memoria **EEPROM externa (24LC256). | 🟡 | Componente adquirido, todavia no se empezo a implementar. |
| 3.3 | Carga y validación de integridad de los parámetros guardados al iniciar el sistema. | 🟡 | Todavia no implemente la EEPROM para empezar con eso. |

#### **2.4 Conectividad Inalámbrica (Bluetooth HM-10)**

| Req ID | Descripción | Estado | Observaciones |
| :---: | :--- | :---: | :--- |
| 4.1 | Envío periódico de tramas UART hacia el módulo **HM-10** con las lecturas de gases, temperatura y humedad. | 🟡 | Componente adquirido, todavia no se empezo a implementar. |
| 4.2 | Recepción y procesamiento de comandos desde la App móvil para modificar umbrales remotamente. | 🟡 | Todavia no se empezo a desarrollar la app móvil. |

#### **2.5 Arquitectura de Firmware, Modos y Hardware Físico**

| Req ID | Descripción | Estado | Observaciones |
| :---: | :--- | :---: | :--- |
| 5.1 | Implementación de modos de operación `NORMAL`, `SET_UP`, `FALLA` y sub-estado `EMERGENCIA_CO`. | 🟡 | Diagrama de estados completo; codificación modular en progreso. |
| 5.2 | Detección de fallas de hardware (desconexión $\text{I}^2\text{C}$ o ADC fuera de rango) con transición a estado seguro. | 🟡 | Verificación de *chip ID* del BME280 implementada; falta manejo de *timeouts* en bus. |
| 5.3 | Arquitectura *Bare Metal* (*Event-Triggered*), *Super-Loop*, *SysTick* y periféricos no bloqueantes (`_IT` / `_DMA`). | 🟡 | ADC+DMA ya opera sin bloqueo; falta migrar la lectura del BME280 de *polling* corto a modo interrupción (`HAL_I2C_Mem_Read_IT`). |
| 5.4 | Implementación de modo de bajo consumo (*Sleep Mode*) entre intervalos de muestreo. | 🟡 | Se implementará una vez estabilizado el *Super-Loop* completo. |
| 5.5 | Montaje definitivo en PCB o placa experimental perforada con componentes y cables soldados (sin protoboard ni Dupont). | 🟡 | Todavia no termine de probar la implementacion de todos los componentes en el protoboard. |

---
