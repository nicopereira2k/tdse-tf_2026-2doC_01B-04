# **Estación de Monitoreo de Calidad de Aire, Clima y Detección de Monóxido de Carbono (EMCAD-CO)**

**Autores:** Pereira Nicolás Manuel

**Padrones:** 105269

**Curso - Grupo:** 01B - Grupo 04

**Fecha:** 2do Cuatrimestre 2026

---

### **1. Selección del proyecto a implementar**

#### **1.1 Objetivo del proyecto y resultados esperados**

El objetivo principal de este proyecto es diseñar e implementar un prototipo embebido de escritorio sobre la plataforma **STM32** para supervisar de forma continua la calidad del aire ambiente, el confort térmico y la presencia de monóxido de carbono ($\text{CO}$) en interiores (oficinas, hogares o laboratorios). 

El sistema utilizará el periférico **ADC en modo Scan con DMA** para adquirir simultáneamente señales de dos sensores analógicos: la concentración de monóxido de carbono ($\text{CO}$) mediante un sensor MQ-7 y la calidad global del aire/gases orgánicos ($\text{VOCs}/\text{CO}_2$) mediante un sensor MQ-135. Asimismo, integrará la medición de temperatura y humedad ambiente a través de un bus digital I2C.

Como objetivos adicionales, el sistema incluirá:

* **Priorización estricta de seguridad:** Jerarquización de alarmas en la arquitectura de firmware, otorgando máxima prioridad a la detección de $\text{CO}$ ($> 50\text{ ppm}$) con patrones sonoros continuos de evacuación e inhabilitación del silenciado manual mientras persista la condición de peligro.
* **Ajuste y persistencia de parámetros:** Un menú interactivo local y soporte para almacenamiento en memoria **EEPROM externa I2C**, guardando los umbrales de alerta de $\text{CO}$ y $\text{CO}_2$, frecuencias de muestreo y constantes de calibración de manera no volátil.
* **Sistemas de indicación gradual:** Semáforo LED de tres estados (Verde = Bueno, Amarillo = Moderado, Rojo = Alerta/Peligro) modulado por PWM y **Buzzer piezoeléctrico** gestionado sin bloqueos temporales.
* **Conectividad inalámbrica:** Módulo **Bluetooth HM-10 (UART)** gestionado por interrupción/DMA para transmitir en tiempo real la trama extendida de datos hacia una App móvil y permitir la configuración remota.
* **Gestión de modos y consumo energético:** Implementación de modos de operación `NORMAL`, `SET_UP`, `FALLA` y `EMERGENCIA_CO`, reduciendo el consumo energético mediante rutinas *Sleep mode* entre intervalos de sensado.
* **Entradas físicas directas:** **DIP switches** para habilitar o deshabilitar físicamente el audio secundario y seleccionar el modo de ahorro de energía.

El firmware utilizará una arquitectura **Event-Triggered System** sin Sistema Operativo (Bare Metal), basada en un **Ejecutivo Cíclico (Super-Loop)** controlado por **SysTick (1 ms)**, procesamiento no bloqueante y máquinas de estado finitas (FSM). Al finalizar el cuatrimestre se entregará un Producto Mínimo Viable (PMV) montado sobre una placa base soldada (sin protoboard ni cables Dupont).

---

#### **1.2 Proyectos similares**

Se evaluaron tres alternativas tecnológicas para llevar a cabo el desarrollo del monitor ambiental y de seguridad:

1. **Estación Avanzada con Detección de $\text{CO}$ en STM32:** Incluye sensor analógico multicanal de $\text{CO}$ (MQ-7) y gas/gases orgánicos (MQ-135) por ADC con DMA, sensor de temperatura/humedad I2C (SHT30/BME280), pantalla LCD/OLED I2C, memoria EEPROM I2C externa, módulo Bluetooth HM-10, semáforo LED PWM, buzzer y DIP switches.
2. **Estación Básica de Calidad de Aire en STM32:** Medición analógica de un único gas (MQ-135) con lectura en semáforo LED y buzzer, sin detección de $\text{CO}$, sin EEPROM externa y sin interfaz Bluetooth.
3. **Monitor Standalone Básico de Gas:** Dispositivo minimalista de seguridad que sensa un único canal analógico de gas y activa directamente un relé/buzzer al superar un umbral analógico prefijado. No posee pantalla LCD, menú interactivo, memoria EEPROM ni conectividad inalámbrica BLE..

Para seleccionar la alternativa óptima, se definieron seis criterios ponderados del 1 al 10:

1. **Disponibilidad de Hardware (peso 9):** Facilidad de adquisición local de los componentes y sensores.
2. **Calidad de la experiencia y Seguridad (peso 8):** Nivel de funcionalidad del dispositivo (monitoreo ambiental + seguridad humana contra $\text{CO}$, conectividad BLE, gráficos y persistencia).
3. **Escalabilidad del sistema (peso 7):** Capacidad para incorporar nuevos actuadores (ej. relé para extractor de aire) o sensores adicionales.
4. **Dificultad técnica / Viabilidad (peso 10):** Cumplimiento de los requisitos del curso (ADC multicanal con DMA, I2C, UART, FSM, bajo consumo, Bare-Metal).
5. **Costo (peso 6):** Presupuesto total requerido para la integración de hardware.
6. **Interés personal del equipo (peso 8):** Motivación por diseñar un dispositivo de seguridad crítica funcional e integrado.

La Tabla 1.2.1 presenta la matriz de decisión y ponderación de las alternativas.

| Criterio | Estación Avanzada $\text{CO}$/Aire STM32 | Estación Básica STM32 | Detector Básico Arduino UNO |
| :--- | :---: | :---: | :---: |
| **Disponibilidad de Hardware** (peso: 9) | 9 (81) | 10 (90) | 10 (90) |
| **Calidad y Seguridad** (peso: 8) | 10 (80) | 6 (48) | 5 (40) |
| **Escalabilidad** (peso: 7) | 10 (70) | 6 (42) | 4 (28) |
| **Dificultad técnica / Viabilidad** (peso: 10) | 10 (100) | 7 (70) | 3 (30) |
| **Costo** (peso: 6) | 7 (42) | 9 (54) | 9 (54) |
| **Interés personal** (peso: 8) | 10 (80) | 7 (56) | 5 (40) |
| **Puntaje Total Ponderado** | **453** | **360** | **282** |

*Tabla 1.2.1: Comparación y ponderación de alternativas de proyecto.*

---

#### **1.3 Selección de proyecto**

Basado en la evaluación de la Tabla 1.2.1, se selecciona la **Estación Avanzada con Detección de $\text{CO}$ y Calidad de Aire en STM32** (Puntaje: 453).

Esta alternativa no solo abarca la totalidad del hardware obligatorio exigido por el curso, sino que potencia la exigencia técnica del firmware al implementar la adquisición del periférico ADC en modo Scan gestionado por DMA para dos canales de sensado analógico ($\text{CO}$ y $\text{VOCs}$/Aire). Asimismo, fuerza la implementación de una máquina de estados con sub-estados de emergencia e interrupciones no bloqueantes, superando las limitaciones de un desarrollo básico.

###### **1.3.1 Diagrama en bloques del sistema**

En la Figura 1.3.1 se representa la estructura de hardware del sistema:

```
                      +-----------------------------------+
                      |         Alimentación DC           |
                      |          (5 V / 3,3 V)            |
                      +-----------------+-----------------+
                                        |
                                        v
+-----------------------+     +-------------------+     +-----------------------+
|  Sensores Analógicos  |     |                   |     | Interfaz del Usuario  |
| - Gas CO (MQ-7 ADC1)  |---->|                   |<----| - 3x Pulsadores (GPIO)|
| - Aire (MQ-135 ADC2)  |---->|                   |     | - DIP Switches (GPIO) |
+-----------------------+     |                   |     +-----------------------+
 (Muestreo via DMA)           |    Micro-         |
                              |    controlador    |     +-----------------------+
+-----------------------+     |    STM32          |     | Indicadores y Alarmas |
| Periféricos Digitales |     |   (Cortex-M)      |---->| - Semáforo LEDs (PWM) |
| - Temp/Hum (SHT30 I2C)|<===>|                   |---->| - Buzzer Piezo (PWM)  |
| - LCD 16x2 / OLED I2C |<===>|                   |     +-----------------------+
| - EEPROM 24LC256 I2C  |<=== |                   |
+-----------------------+     |                   |     +-----------------------+
                              |                   |<===>| Módulo de Comunicación|
                              |                   |     | - HM-10 BLE (UART)    |
                              +-------------------+     +-----------------------+
```

*Figura 1.3.1: Diagrama en bloques del hardware del sistema EMCAD-CO.*

---

### **2. Elicitación de requisitos y casos de uso**

En el mercado conviven detectores dedicados de monóxido de carbono (de accionamiento doméstico autónomo) con monitores de calidad de aire comercializados para oficinas. Sin embargo, los detectores de $\text{CO}$ residenciales carecen de conectividad, pantallas avanzadas o sensado climático, mientras que los monitores de calidad de aire comerciales rara vez incorporan detección de emergencias por toxicidad de $\text{CO}$.

El desarrollo propuesto integra en una única plataforma embebida ambas funcionalidades, combinando la medición continua de confort con un sistema robusto de seguridad contra gases peligrosos, manteniendo la transparencia total del código fuente y permitiendo el control total de parámetros a nivel industrial o académico.

#### **2.1 Requisitos del proyecto**

En la Tabla 2.1 se enumeran los requisitos funcionales organizados por área:

| Grupo | ID | Descripción | Categoría |
| :--- | :--- | :--- | :--- |
| **Sensado Analógico** | 1.1 | El sistema leerá el canal ADC1 (MQ-7) correspondiente a la concentración de $\text{CO}$ mediante Scan Mode y DMA. | Hardware Obligatorio |
| | 1.2 | El sistema leerá el canal ADC2 (MQ-135) correspondiente a gases orgánicos/calidad de aire en modo DMA. | Hardware Obligatorio |
| | 1.3 | El sistema aplicará un filtro por media móvil digital a cada canal analógico para atenuar ruido. | Programación Obligatoria |
| **Sensado Digital** | 1.4 | El sistema leerá la temperatura y humedad ambiente desde un sensor I2C a intervalos regulares. | Hardware Adicional |
| **Actuación e Interfaz**| 2.1 | El sistema indicará mediante un semáforo LED el nivel de calidad del aire y la presencia de $\text{CO}$. | Hardware Obligatorio |
| | 2.2 | El sistema modulará el brillo de los LEDs por PWM según el nivel de advertencia determinado. | Programación Obligatoria |
| | 2.3 | El sistema activará una alarma sonora no bloqueante discontinua ante mala calidad de aire, y continua de alta prioridad ante detección de $\text{CO}$ ($> 50\text{ ppm}$). | Hardware Obligatorio |
| | 2.4 | El sistema contará con 3 pulsadores con antirrebote por software para navegar y configurar el menú. | Hardware Obligatorio |
| | 2.5 | El sistema incluirá DIP switches para habilitar/deshabilitar alertas secundarias y conmutar el modo de energía. | Hardware Obligatorio |
| | 2.6 | El sistema mostrará las mediciones ($\text{PPM}_{\text{CO}}$, $\text{PPM}_{\text{Aire}}$, Temp, Hum) y el estado en la pantalla LCD. | Hardware Adicional |
| **Persistencia y Menú** | 3.1 | El sistema guardará los umbrales de alerta de $\text{CO}$, $\text{CO}_2$ y constantes de calibración en EEPROM I2C externa. | Hardware Obligatorio |
| | 3.2 | El sistema cargará los datos de la EEPROM durante la secuencia de inicialización del firmware. | Programación Obligatoria |
| | 3.3 | El sistema implementará un menú interactivo navegable mediante la botonera local. | Programación Obligatoria |
| **Comunicación BLE** | 4.1 | El sistema enviará tramas periódicas formateadas por UART hacia el módulo HM-10 utilizando callbacks. | Hardware Obligatorio |
| | 4.2 | El sistema procesará comandos remotos recibidos por Bluetooth para modificar parámetros o forzar el test de alarmas. | Programación Obligatoria |
| **Arquitectura de Software**| 5.1 | El firmware operará bajo estructura Bare Metal con SysTick a $1\text{ ms}$ y bucle principal $< 1\text{ ms}$. | Programación Obligatoria |
| | 5.2 | El sistema gestionará la lógica mediante máquinas de estado finitas (FSM) con modos `NORMAL`, `SET_UP`, `FALLA` y `EMERGENCIA_CO`. | Programación Obligatoria |
| | 5.3 | En caso de error de lectura analógica o pérdida de comunicación I2C, el sistema ingresará al estado `FALLA` colocando las salidas en estado seguro. | Programación Obligatoria |

*Tabla 2.1: Requisitos del proyecto.*

---

#### **2.2 Caso de uso 1: Ocurrencia de alerta crítica por Detección de Monóxido de Carbono ($\text{CO}$)**

| Elemento | Definición |
| :--- | :--- |
| **Disparador** | La concentración de $\text{CO}$ medida por el sensor MQ-7 supera el umbral crítico de $50\text{ ppm}$ guardado en EEPROM. |
| **Precondiciones** | El sistema se encuentra encendido, operando en modo `NORMAL`, con el periférico ADC transfiriendo muestras mediante DMA en segundo plano. |
| **Flujo principal** | 1. La FSM detecta en la tarea de procesamiento analógico que $\text{PPM}_{\text{CO}} > 50\text{ ppm}$.<br>2. El sistema interrumpe el flujo normal y conmuta al estado de máxima prioridad `EMERGENCIA_CO`.<br>3. Se enciende de forma parpadeante el LED Rojo y se activa el Buzzer en modo continuo de alta frecuencia.<br>4. Se muestra en la pantalla LCD la leyenda: `¡ALERTA CRÍTICA! - CO DETECTADO`.<br>5. Se transmite de inmediato una trama de emergencia por el módulo Bluetooth HM-10 a la aplicación móvil.<br>6. El sistema permanece en este estado hasta que la concentración de $\text{CO}$ descienda por debajo de los $35\text{ ppm}$. |
| **Flujos alternativos**| **a. Intento de silenciado manual:** Si el usuario presiona el botón *Silenciar Alarma*, el firmware emite un beep de rechazo y mantiene la sirena y los LEDs de emergencia activos mientras persista el gas tóxico.<br>**b. Fallo en transmisión BLE:** Si la comunicación por UART falla, la alarma local sonora y visual continúa operando sin interrupción. |

*Tabla 2.2: Caso de uso 1: Detección crítica de Monóxido de Carbono.*

---

#### **2.3 Caso de uso 2: Configuración de umbrales en EEPROM mediante el Menú Local**

| Elemento | Definición |
| :--- | :--- |
| **Disparador** | El usuario presiona la tecla *Menú/Modo* en el panel frontal del dispositivo. |
| **Precondiciones** | El sistema está encendido en estado `NORMAL` y no existen condiciones activas de emergencia por $\text{CO}$. |
| **Flujo principal** | 1. La FSM conmuta del estado `NORMAL` al estado `SET_UP`.<br>2. Las lecturas de los sensores analógicos continúan realizándose en segundo plano por DMA para garantizar la seguridad.<br>3. El display muestra la pantalla de ajuste de umbral: `Umbral CO: [50] ppm`.<br>4. El usuario utiliza el botón *Incrementar* para ajustar el valor deseado y presiona *Aceptar*.<br>5. El firmware escribe la nueva configuración en la memoria EEPROM externa vía bus I2C mediante una rutina no bloqueante.<br>6. El sistema confirma la escritura con un doble beep corto y regresa al modo `NORMAL`. |
| **Flujos alternativos**| **a. Timeout por inactividad:** Si transcurren 15 segundos sin presionar ningún botón, el sistema descarta los cambios pendientes y vuelve automáticamente a `NORMAL`.<br>**b. Error de comunicación I2C:** Si la memoria EEPROM no responde durante la escritura, el sistema muestra `ERR: EEPROM WRITE` y transiciona al modo `FALLA`. |

*Tabla 2.3: Caso de uso 2: Configuración de umbrales mediante menú.*

---

#### **2.4 Caso de uso 3: Sincronización y monitoreo remoto vía App Bluetooth (HM-10)**

| Elemento | Definición |
| :--- | :--- |
| **Disparador** | Cumplimiento del período de envío telemétrico (cada $2\text{ s}$) o recepción de un comando de lectura desde la App móvil. |
| **Precondiciones** | El módulo HM-10 se encuentra energizado y enlazado vía Bluetooth con el dispositivo móvil. |
| **Flujo principal** | 1. La tarea temporizada de comunicación empaqueta las últimas lecturas filtradas ($\text{CO}$, $\text{Aire}$, Temp, Hum, Estado) en un *buffer* de texto.<br>2. El firmware inicia la transmisión de la trama UART hacia el módulo HM-10 utilizando interrupciones/DMA sin bloquear el bucle principal.<br>3. La App móvil recibe la trama, parsea los valores y actualiza los indicadores en la interfaz gráfica.<br>4. Al finalizar la transmisión, el callback de la UART libera la estructura de datos para el siguiente ciclo. |
| **Flujos alternativos**| **a. Recepción de comando de calibración:** Si la App envía la trama `SET_CO_THRS=40`, el firmware valida los parámetros y actualiza el umbral en la memoria EEPROM, respondiendo con una confirmación `OK`. |

*Tabla 2.4: Caso de uso 3: Monitoreo remoto por Bluetooth.*
