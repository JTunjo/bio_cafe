# Solicitud de Cotización (RFQ)
## Propagador de Cultivos Iniciadores para Fermentación de Café de Especialidad

**Versión:** Propagador — equipo complementario al Biorreactor Ultra  
**Aplicación:** Propagación aerobia de levaduras y bacterias lácticas para uso como inóculo en fermentaciones SIAF  
**Nivel:** Funcional, robusto, de bajo mantenimiento

---

## 1. Contexto y objetivo

Este equipo opera en paralelo al biorreactor principal (Biorreactor Ultra) y tiene un único propósito: **producir inóculos viables, concentrados y de composición conocida** para ser dosificados en el reactor de fermentación.

Los cultivos objetivo son principalmente:

- *Saccharomyces cerevisiae* y otras levaduras de interés enológico/fermentativo
- Bacterias ácido-lácticas (LAB): *Lactiplantibacillus plantarum* y afines
- Consorcios mixtos definidos por el usuario

El propagador no fermenta café. Su sustrato es un medio de cultivo líquido definido (melaza diluida, mosto o medio sintético). El diseño debe priorizar **simplicidad operativa, limpieza efectiva y confiabilidad de las condiciones de cultivo** sobre cualquier otro atributo.

---

## 2. Alcance del suministro

- Diseño y fabricación del sistema completo
- Sistema de control (hardware + firmware)
- Sensores e instrumentación según especificación
- Instalación, puesta en marcha y capacitación
- Documentación técnica y operativa
- Soporte postventa mínimo 2 años

---

## 3. Capacidad y volumen de trabajo

- **Volumen de trabajo:** 20–100 L (el proveedor debe justificar el volumen propuesto en función de la relación inóculo:reactor principal de 1–5% v/v)
- Vessel de acero inoxidable AISI 316L, pulido sanitario interior Ra ≤ 0.8 µm
- Fondo cónico para drenaje completo sin residuos
- El proveedor debe indicar el volumen mínimo y máximo de trabajo operativo

---

## 4. Instrumentación

El propagador requiere exactamente cuatro variables. No más.

| Variable | Justificación | Precisión requerida | Frecuencia mínima |
|---|---|---|---|
| **Temperatura** | Control del rango óptimo de crecimiento por especie | ±0.5 °C | 60 s |
| **pH** | Indicador de avance de cultivo y salud del consorcio | ±0.05 pH | 60 s |
| **Oxígeno disuelto (DO)** | Variable de control primaria en propagación aerobia; determina tasa de crecimiento y rendimiento de biomasa | ±0.5 mg/L | 30 s |
| **Presión** | Seguridad y verificación de integridad del sistema de aireación | ±0.05 bar | 60 s |

Los sensores deben ser de grado industrial, cuerpo AISI 316L, compatibles con CIP. Salida digital preferida (Modbus o equivalente).

**Sobre el DO:** a diferencia del biorreactor SIAF, aquí el DO es la variable de control activa durante todo el proceso. La tasa de aireación se regula para mantener el DO en el rango óptimo de la especie en cultivo (típicamente 20–80% de saturación). Es el sensor más importante del sistema.

### 4.2 Arquitectura de medición: inmersión directa obligatoria

**Todos los sensores del propagador deben estar instalados directamente en el vessel, en inmersión en el medio de cultivo. No se acepta arquitectura de cámara de muestreo externa.**

La razón es funcional: el control de DO en lazo cerrado requiere una respuesta en tiempo real sin retardo. Cualquier circuito de extracción externa introduce una latencia que degrada la calidad del control, especialmente en la fase exponencial de crecimiento donde el consumo de oxígeno cambia rápidamente. En un medio líquido homogéneo como el usado en propagación, la inmersión directa no presenta las desventajas que justificarían una cámara de muestreo en el biorreactor de fermentación.

El proveedor debe especificar el procedimiento de calibración de cada sensor en inmersión, incluyendo acceso físico sin desmontaje del vessel.

---

## 5. Control de proceso

### 5.1 Temperatura

- Calentamiento y enfriamiento activos (camisa o serpentín)
- Rango operativo: 20–40 °C
- Sin requisito de rampa rápida de enfriamiento — el arresto de propagación es por tiempo, no por temperatura de emergencia

### 5.2 Aireación y oxigenación

- Sistema de aireación con compresor o línea de aire estéril filtrado (filtro 0.2 µm)
- Difusor de burbuja fina (sparger) para maximizar transferencia de oxígeno
- Caudal de aire ajustable y medido (caudalímetro másico o rotámetro calibrado)
- Control de DO en lazo cerrado: el sistema debe ajustar caudal de aire o velocidad de agitación para mantener setpoint de DO configurado en receta
- **Sin inyección de O₂ puro** como estándar — aire comprimido estéril es suficiente para los cultivos objetivo

### 5.3 Agitación

- Agitador mecánico de eje superior o inferior, con impulsor tipo Rushton o marina
- Velocidad configurable: rango mínimo 50–500 RPM
- La agitación es el segundo actuador para control de DO (junto con caudal de aire)
- Lectura de RPM real (encoder o equivalente), no solo setpoint

### 5.4 pH

- Medición continua
- **Sin control activo de pH** como requerimiento estándar — la corrección de pH en propagación de inóculos para café se hace manualmente antes de inocular el reactor principal
- El sistema debe emitir alerta si pH sale de rango configurado

### 5.5 Estimación de densidad celular y proyección de tiempo de cosecha

El sistema debe incluir un **modelo cinético de crecimiento en línea** que permita al operador conocer en tiempo real el estado del cultivo y recibir una proyección del momento óptimo de transferencia al reactor principal.

**Parámetros de entrada por ciclo (ingresados por el operador al inicio):**

- Densidad celular inicial estimada (células viables/L), obtenida por conteo externo (hemocitómetro, citometría de flujo u otro método de referencia del usuario)
- Densidad celular objetivo para transferencia (células viables/L)
- Especie o cepa en cultivo (seleccionable desde biblioteca configurable)

**Funcionamiento del modelo:**

El sistema debe estimar la tasa de crecimiento específica (µ, h⁻¹) de forma continua a partir de las variables medidas, utilizando como proxies de actividad metabólica:

- Tasa de consumo de oxígeno disuelto (dDO/dt) — indicador primario
- Evolución de pH — indicador secundario
- Temperatura — factor corrector del modelo

A partir de µ estimado en tiempo real, el sistema calcula:

- **Densidad celular estimada actual** (modelo exponencial: N(t) = N₀ · e^(µ·t))
- **Tiempo estimado para alcanzar densidad objetivo**
- **Confianza de la proyección**, expresada como rango (ej. "entre 1h 30min y 2h 00min")

**Requisitos de implementación:**

- El modelo debe ser **calibrable por el usuario**: el sistema debe permitir ingresar resultados de conteos externos durante o después del ciclo para ajustar los parámetros del modelo a las condiciones reales de la finca (cepa, medio, temperatura)
- El sistema debe mantener un **historial de ciclos con conteos validados** que permita refinar el modelo con el tiempo — el sistema aprende de cada ciclo en que el usuario provee un conteo de referencia
- La biblioteca de cepas debe ser editable: el usuario puede crear perfiles con µ máximo (µ_max) y parámetros de Monod específicos para las cepas que utiliza
- La proyección debe actualizarse **cada 5 minutos** como mínimo y mostrarse de forma prominente en la HMI

**Alerta de cosecha:**

- El sistema debe emitir una alerta visual y sonora cuando la densidad estimada alcance el **90% del objetivo**, dando tiempo al operador para preparar la transferencia
- La alerta debe incluir: densidad estimada actual, tiempo estimado restante, y recomendación de confirmar con conteo externo si el lote es crítico

**Limitación explícita a documentar:**

El proveedor debe declarar claramente en su propuesta que este es un **modelo estimativo basado en proxies indirectos**, no un conteo directo. La precisión del modelo depende de la calidad de los datos de calibración acumulados. Se recomienda al usuario validar con conteo externo los primeros 5–10 ciclos de cada cepa nueva antes de confiar en la proyección como único criterio de transferencia.

### 5.5 Transferencia del inóculo al reactor principal

- Puerto de descarga aséptico compatible con la línea de inoculación del Biorreactor Ultra
- Drenaje completo del volumen de trabajo sin residuos en fondo
- El proveedor debe describir el procedimiento de transferencia aséptica

---

## 6. Recetas y control

Las recetas de propagación son simples. El sistema debe soportar:

- Definición de: temperatura objetivo, setpoint de DO, velocidad de agitación inicial, duración máxima
- Alerta por fin de tiempo o por DO sostenido en meseta (indicador de que el cultivo alcanzó fase estacionaria)
- Registro continuo de las cuatro variables durante todo el ciclo
- **No se requiere lógica event-driven compleja** — la propagación es un proceso más predecible que la fermentación de café

---

## 7. Datos

- Registro continuo a las frecuencias definidas en sección 4
- Buffer local para mínimo 90 días de datos
- Exportación por ciclo: serie temporal completa en CSV o JSON
- ID de ciclo vinculable al ID de lote del reactor principal al que se destinó el inóculo
- API REST o exportación manual — no se requiere MQTT en esta versión

**Trazabilidad mínima del inóculo exportado:**

```json
{
  "inoculum_id": "string",
  "strain_id": "string",
  "propagation_cycle_id": "string",
  "harvest_timestamp": "ISO 8601",
  "harvest_volume_L": float,
  "final_pH": float,
  "final_DO_pct_saturation": float,
  "duration_h": float,
  "destination_batch_id": "string"
}
```

Este registro debe adjuntarse automáticamente al reporte del lote del reactor principal.

---

## 8. Limpieza y mantenimiento

### 8.1 CIP

- Ciclos de limpieza sin desmontaje completo
- Cobertura de: vessel, líneas, sparger, sensores
- Drenaje completo verificable
- El sparger debe ser desmontable para inspección y limpieza manual periódica

### 8.2 Esterilización

- El sistema debe soportar **esterilización por vapor (SIP)** o **autoclavado** de componentes en contacto con el cultivo, dado que la contaminación del inóculo se trasladaría al reactor principal
- Alternativamente, el proveedor puede proponer un protocolo de sanitización química validado (ej. NaOH + ácido peracético) como sustituto del SIP, justificando su equivalencia microbiológica

### 8.3 Filtro de aire

- Filtro estéril en línea de entrada de aire (0.2 µm), con indicador de diferencial de presión
- El proveedor debe especificar frecuencia de reemplazo y disponibilidad en Colombia

---

## 9. Infraestructura y condiciones de operación

- Temperatura ambiente: 15–35 °C
- Equipo de menor exigencia ambiental que el reactor principal — puede ubicarse en sala de preparación o laboratorio básico de finca
- Protección del gabinete de control: IP54 mínimo (no requiere IP65 si opera en interior)
- Suministro eléctrico: el proveedor debe especificar requerimientos
- Suministro de aire: compresor incluido en el alcance o especificación de compresor externo requerido

---

## 10. Información requerida al proveedor

1. Justificación del volumen de trabajo propuesto en relación al reactor principal
2. Ficha técnica de sensores (especialmente sensor de DO: principio de medición, membrana, vida útil en medio con levaduras)
3. Descripción del sistema de control de DO en lazo cerrado
4. Protocolo de esterilización o sanitización propuesto y su validación
5. Procedimiento de transferencia aséptica del inóculo
6. Compatibilidad con puerto de inoculación del Biorreactor Ultra
7. Plan de mantenimiento: frecuencia, componentes críticos, disponibilidad de repuestos en Colombia
8. Costos detallados
9. Tiempo de entrega

---

## 11. Criterios de evaluación

| Criterio | Peso |
|---|---|
| Confiabilidad del control de DO en lazo cerrado | 30% |
| Efectividad y simplicidad del protocolo de esterilización/sanitización | 25% |
| Trazabilidad del inóculo vinculada al lote del reactor principal | 20% |
| Facilidad de limpieza y mantenimiento en contexto de finca | 15% |
| Costo-beneficio | 10% |
