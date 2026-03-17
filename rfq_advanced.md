# Solicitud de Cotización (RFQ)
## Biorreactor Avanzado para Fermentación Controlada de Café de Especialidad

**Versión:** Advanced  
**Aplicación:** Poscosecha — café cereza y despulpado  
**Nivel:** Competitivo en mercado de cafés especiales (costo medio-alto)

---

## 1. Contexto y objetivo

Se solicita cotización para el diseño, fabricación e implementación de un sistema de biorreactor para fermentación controlada de café en condiciones de finca. El objetivo es producir lotes reproducibles con perfil fermentativo documentado, orientados a café de especialidad con puntaje SCA ≥ 84 puntos.

El sistema debe funcionar como **plataforma de ejecución de recetas y adquisición de datos**, operable por personal de finca sin formación técnica especializada, con conectividad a sistemas externos de análisis y trazabilidad.

---

## 2. Alcance del suministro

Se espera suministrar:

- Diseño y fabricación del sistema completo
- Sistema de control (hardware + firmware)
- Sensores e instrumentación según especificación
- Sistema de adquisición y transmisión de datos
- Instalación, puesta en marcha y capacitación
- Documentación técnica y operativa
- Soporte postventa con cobertura mínima de 2 años

---

## 3. Modos de operación requeridos

El sistema debe soportar sin reconfiguraciones físicas:

- **Fermentación en estado sólido (SSF)** — sustrato sin adición de agua libre
- **Fermentación en estado sólido con humectación controlada** — adición medida de agua o lixiviado
- **Fermentación sumergida** — sustrato cubierto con líquido (hasta 30% v/v)
- **SIAF (Self-Induced Anaerobic Fermentation)** — atmósfera anaerobia autogenerada, con hermeticidad verificable y válvula de alivio unidireccional calibrada que permite salida de CO₂ sin entrada de O₂

El sistema debe permitir transiciones entre modos dentro del mismo lote según receta definida.

---

## 4. Instrumentación y control de proceso

### 4.1 Variables medidas y controladas

| Variable | Modo | Rango recomendado | Precisión mínima |
|---|---|---|---|
| Temperatura (sustrato o lixiviado) | Medición + control activo | 10–50 °C | ±0.5 °C |
| pH (lixiviado) | Medición continua | 3.0–7.0 | ±0.05 pH |
| Presión en headspace | Medición continua | 0–2 bar | ±0.02 bar |
| CO₂ en headspace | Medición continua | 0–100% v/v | ±1% absoluto |
| Potencial redox — ORP | Medición continua | -500 a +500 mV | ±5 mV |
| Oxígeno disuelto — DO | Medición continua | 0–20 mg/L | ±0.2 mg/L |

**Nota sobre temperatura:** se requiere al menos **dos puntos de medición independientes** (zona superior e inferior del biorreactor) para verificar uniformidad térmica. El gradiente máximo aceptable entre zonas es ±1.5 °C durante operación estable.

### 4.2 Arquitectura de medición: inmersión directa vs. cámara de muestreo

Se solicita proponer y justificar la arquitectura de medición para pH, ORP y DO. Se aceptan dos arquitecturas:

**Opción A — Inmersión directa:** sensores instalados directamente en el vessel, en contacto con el sustrato o lixiviado.

**Opción B — Cámara de muestreo externa (flow cell):** el lixiviado se extrae continuamente mediante bomba peristáltica hacia una cámara externa donde están los sensores, y se reincorpora al vessel de forma continua y cuantificada.

Si se propone la Opción B, se espera poder definir:

- Retardo de medición entre condición real en vessel y lectura del sensor (en segundos o minutos)
- Diseño del circuito de reincorporación y su impacto sobre el balance de masa y la integridad de la presión del headspace en modo SIAF
- Caudal de extracción y reincorporación, y gestión en SSF donde el lixiviado disponible puede ser limitado
- Procedimiento de limpieza CIP de la cámara y las líneas de muestreo
- Impacto sobre frecuencia de calibración comparado con inmersión directa
- Para el sensor de DO específicamente: ¿Se puede garantizar que el retardo de la cámara no compromete el control en lazo cerrado si DO es utilizado como variable de control activa?

La temperatura, la presión del headspace y el CO₂ en headspace se miden siempre mediante sensores instalados directamente en el vessel o en la línea de gas, independientemente de la arquitectura elegida para los demás sensores.

### 4.2 Control de atmósfera

- Inyección de CO₂ para establecimiento de atmósfera anaerobia inicial
- Inyección de N₂ como gas de purga o inertización
- Válvula de alivio de presión unidireccional (airlock de precisión), configurable en set-point
- **No se requiere inyección de O₂** en esta versión

### 4.3 Control térmico

- Calentamiento y enfriamiento activos (camisa o serpentín)
- Rampa de enfriamiento para arresto de fermentación: mínimo **0.5 °C/min** hasta temperatura objetivo
- Preacondicionamiento térmico previo a carga del lote

### 4.4 Mezcla y recirculación

- Agitación de **baja cizalla** para SSF (volteo periódico o paletas), sin daño a estructura del grano
- Recirculación de lixiviados mediante bomba peristáltica con caudal ajustable
- Intensidad de mezcla configurable por etapa de receta
- Operación en modo estático (sin agitación) como opción válida para SSF puro

### 4.5 Inoculación de cultivos iniciadores

- Puerto de inoculación aséptico, sin ruptura de hermeticidad ni de atmósfera
- Dosificación volumétrica controlada (bomba peristáltica o equivalente)
- Soporte para **inoculación secuencial**: dos puertos independientes o protocolo de dos tiempos (ej. LAB en t=0, *Saccharomyces* en t=X según receta)
- Registro automático: ID de inóculo, volumen dosificado, timestamp

---

## 5. Gestión de datos

### 5.1 Adquisición

- Registro continuo de todas las variables instrumentadas
- Frecuencia configurable por variable: mínimo cada **60 segundos** para CO₂ y ORP; cada **5 minutos** para las demás
- Marca de tiempo con sincronización NTP

### 5.2 Estructura de registro por lote

Cada registro debe incluir:

```json
{
  "batch_id": "string",
  "timestamp": "ISO 8601",
  "sensors": {
    "temperature_top_C": float,
    "temperature_bottom_C": float,
    "pH": float,
    "pressure_bar": float,
    "CO2_pct": float,
    "ORP_mV": float,
    "DO_mg_L": float
  },
  "process_state": {
    "mode": "SSF | submerged | SIAF",
    "stage_name": "string",
    "stage_elapsed_min": int
  },
  "actuators": {
    "heating": bool,
    "cooling": bool,
    "agitation_rpm": float,
    "recirculation_L_min": float
  }
}
```

### 5.3 Almacenamiento y resiliencia

- Buffer local con capacidad mínima para **30 días** de datos a máxima frecuencia de muestreo
- Operación completamente funcional sin conexión a internet
- Sincronización automática al recuperar conectividad
- Garantía de no pérdida de datos ante corte de energía (escritura en almacenamiento no volátil)

### 5.4 Acceso y exportación

- API REST para consulta de datos en tiempo real e histórico
- Consulta histórica por: ID de lote, variable, rango de timestamp

### 5.5 Resumen automático por lote

Al cierre de cada lote, el sistema debe generar automáticamente un reporte que incluya:

- Duración total y por etapa
- Perfil cinético completo (serie temporal exportable) de cada variable
- Estadísticos por etapa: media, máximo, mínimo, desviación estándar
- Registro de eventos: alertas, desvíos de setpoint, acciones del operador
- Registro de inóculos aplicados
- Código QR o identificador único del lote para trazabilidad externa

---

## 6. Ejecución de recetas

### 6.1 Estructura de receta

Las recetas deben definirse en formato estructurado (JSON o YAML) e incluir:

- Nombre, versión y autor de la receta
- Secuencia de etapas, cada una con:
  - Nombre de etapa
  - Duración máxima
  - Setpoints de temperatura, presión, mezcla, recirculación, atmósfera
  - **Condición de avance**: por tiempo, o por variable alcanzando umbral (ej. `pH < 3.8`, `CO2_acumulado > X`)
  - Alertas configurables por desviación de setpoint

### 6.2 Gestión de recetas

- Importación desde fuente externa (archivo o API)
- Biblioteca local de recetas con control de versiones
- Ejecución desde HMI local sin dependencia de nube
- Registro de qué receta (nombre + versión) ejecutó cada lote

### 6.3 Lógica condicional básica

El sistema debe soportar al menos:

- Avance de etapa por condición de variable (event-driven)
- Alerta y pausa automática si variable crítica sale de rango configurado
- Arresto de emergencia con enfriamiento activo activado automáticamente

---

## 7. Interfaz de operación (HMI)

- Pantalla táctil local, mínimo 7"
- Visualización en tiempo real de todas las variables en un solo dashboard
- Gráficas de tendencia de las últimas 24 h visibles sin navegación adicional
- Flujos guiados para: inicio de lote, calibración de sensores, limpieza CIP, arresto manual
- Idioma configurable (español requerido)
- Acceso remoto vía navegador web (red local o VPN) con las mismas capacidades de visualización

---

## 8. Requerimientos operativos

### 8.1 Calibración de sensores

- Acceso físico directo a todos los sensores sin desmontaje del biorreactor
- Protocolos guiados de calibración desde HMI (pH: buffer 2 puntos mínimo; CO₂: gas de referencia certificado)
- Registro automático de cada calibración: fecha, usuario, valores antes/después
- Alertas de recalibración por tiempo o por desvío detectado

### 8.2 Limpieza in-situ (CIP)

- Ciclos de limpieza configurables sin desmontaje completo
- Cobertura de: tanque principal, líneas de recirculación, sensores
- Drenaje completo verificable
- Registro de cada ciclo CIP en log del sistema

### 8.3 Preparación de lote

- Preacondicionamiento térmico previo a carga
- Estabilización de condiciones iniciales antes de inicio de receta
- Puerto de muestreo aséptico de lixiviado para análisis de laboratorio durante el proceso, sin abrir el biorreactor

### 8.4 Post-proceso

- Arresto de fermentación por enfriamiento rápido (≥ 0.5 °C/min)
- Drenaje de lixiviados separado del material sólido
- Descarga eficiente del sustrato sólido

---

## 9. Infraestructura y condiciones de operación

### 9.1 Condiciones ambientales

- Temperatura ambiente: 5–40 °C
- Humedad relativa: 20–95% HR sin condensación en electrónica
- Protección del gabinete de control: **IP65 mínimo**

### 9.2 Energía

- Especificar voltaje y potencia requeridos
- **UPS integrado** con autonomía mínima de **2 horas** para:
  - Mantener adquisición de datos
  - Ejecutar protocolo de arresto controlado ante corte de energía
  - Preservar datos en tránsito

### 9.3 Servicios requeridos

Cuáles serían los valores de:

- Consumo de agua (proceso + CIP)
- Requerimientos de drenaje
- Consumo de gases (CO₂, N₂): presión de suministro y caudal máximo
- Espacio mínimo de instalación (huella + área de trabajo)

---

## 10. Información requerida

La estimación requiere al menos:

1. Descripción de la solución y arquitectura del sistema
2. Especificaciones técnicas completas de sensores y actuadores (marca, modelo, rango, precisión)
3. Descripción de la lógica de control (PID, controladores utilizados)
4. Capacidades de integración de datos (API, MQTT, formatos)
5. Esquema eléctrico simplificado y requerimientos de infraestructura
6. Procedimiento y tiempo estimado de instalación
7. Plan de mantenimiento preventivo: frecuencia, componentes críticos, disponibilidad de repuestos en Colombia
8. Costos detallados (equipo, instalación, capacitación, soporte)
9. Descripción de servicios requeridos (9.3)
10. Tiempo de entrega
