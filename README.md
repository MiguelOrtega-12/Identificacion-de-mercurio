# HgSense AgNPs

**Web App móvil (PWA estática) de colorimetría digital con nanopartículas de plata (AgNPs) para tamizaje rápido en campo de mercurio inorgánico ($\text{Hg}^{2+}$) y especiación de metilmercurio ($\text{CH}_3\text{Hg}^+$) mediante foto-oxidación UV-C (254 nm).**

Diseñada 100% *client-side* en **HTML5, CSS3 y JavaScript Vanilla** (sin frameworks ni backend) para desplegarse directamente en **GitHub Pages** y operar en teléfonos celulares en cuencas mineras y fluviales de Bolivia.

---

## 1. Problema y Propuesta de Valor (Hackathon)

En cuencas amazónicas y andinas impactadas por la minería aurífera artesanal y de pequeña escala (ASGM) —como **Río Beni, Tipuani, Mapiri y Lago Titicaca**—, el monitoreo de mercurio depende de análisis de laboratorio acreditado (**CV-AAS / ICP-MS**) que son costosos, lentos y logísticamente complejos para monitoreo frecuente.

**HgSense AgNPs** resuelve la brecha de tamizaje en terreno con un sistema **"Plug & Play" en 2 Niveles** que convierte la cámara trasera de cualquier smartphone en un colorímetro digital calibrado:

- **Bajo costo por prueba** usando AgNPs amarillas sintetizadas por química verde.
- **Decisión inmediata en campo** mediante semáforo normativo alineado a la legislación boliviana e internacional.
- **Filtro inteligente de especiación:** evita activar la etapa UV-C innecesariamente cuando el tamizaje rápido de Nivel 1 resulta limpio.

---

## 2. Fundamento Químico y Óptico

### 2.1 Detección colorimétrica de $\text{Hg}^{2+}$ con AgNPs amarillas
Las nanopartículas de plata coloidales presentan una banda intensa de Resonancia de Plasmón Superficial Localizado (LSPR) alrededor de **400 nm**, lo que les confiere un color amarillo intenso saturado.

En presencia de iones mercúricos ($\text{Hg}^{2+}$), ocurre una reacción redox espontánea impulsada por la diferencia de potencial estándar ($E^\circ_{\text{Hg}^{2+}/\text{Hg}^0} = +0.85\text{ V} > E^\circ_{\text{Ag}^+/\text{Ag}^0} = +0.80\text{ V}$):

$$2\text{Ag}^0_{(\text{NP})} + \text{Hg}^{2+} \longrightarrow 2\text{Ag}^+ + \text{Hg}^0 \quad (\text{amalgamación Ag-Hg y decoloración})$$

Esta oxidación y amalgamación reduce la intensidad de la banda LSPR, provocando una **pérdida progresiva de saturación del amarillo hacia incoloro** proporcional a la concentración de $\text{Hg}^{2+}$.

### 2.2 Especiación de Metilmercurio ($\text{CH}_3\text{Hg}^+$) por Foto-oxidación UV-C (254 nm)
El $\text{CH}_3\text{Hg}^+$ orgánico no reacciona directamente con la misma cinética que el $\text{Hg}^{2+}$ libre. En el **Nivel 2**, la muestra del **Tubo B (Tapa Morada)** se irradia con luz **UV-C (254 nm)** para fotomineralizar el mercurio orgánico a $\text{Hg}^{2+}$ inorgánico, midiendo luego el **Mercurio Total** en el **Tubo C (Tapa Naranja)**:

$$\text{CH}_3\text{Hg}^+ \xrightarrow{\text{UV-C (254 nm)}} \text{Hg}^{2+}$$

$$[\text{CH}_3\text{Hg}^+] = \max\left(0,\; [\text{Hg}_{\text{total}}] - [\text{Hg}^{2+}]\right)$$

> **Nota de alcance analítico:** La especiación UV-C en campo está diseñada principalmente para **lodos/sedimentos anóxicos (50–500+ ppb)** y **efluentes críticos**; en agua superficial abierta el $\text{CH}_3\text{Hg}^+$ suele encontrarse en el rango de $\text{ng/L}$ y requiere confirmación instrumental en laboratorio (CV-AFS / GC-ICP-MS).

---

## 3. Procesamiento de Imagen y Modelo de Calibración

La aplicación procesa píxeles reales directamente en un elemento **HTML5 `<canvas>`**:

1. Captura la fotografía con `<input type="file" accept="image/*" capture="environment">` (o archivo subido / escenario demo sintético).
2. Dibuja una **Región de Interés (ROI) centrada de 50×50 px** con borde verde punteado (reubicable tocando el lienzo).
3. Extrae los 2,500 píxeles de la ROI vía `ctx.getImageData()` antes de superponer la guía visual, calculando los promedios reales $\bar{R}, \bar{G}, \bar{B}$.
4. Convierte $\text{RGB} \to \text{HSV}$ ($H \in [0^\circ, 360^\circ]$, $S \in [0\%, 100\%]$, $V \in [0\%, 100\%]$) y calcula la razón óptica $B/R$.
5. Aplica el modelo lineal calibrado ($R^2 = 0.992$) frente al **Blanco de Referencia AgNPs** ($\text{RGB}(235, 195, 30) \Rightarrow S_{\text{blanco}} = 87.2\%$):

$$\Delta S = \max(0,\; 87.2 - S_{\text{muestra}})$$

$$C_{\text{extracto}}\;(\text{ppb}) = \Delta S \times 1.45$$

$$C_{\text{final}} = C_{\text{extracto}} \times f_{\text{matriz}}$$

Donde:
- **Agua (Puntos calientes / Efluentes):** $f_{\text{matriz}} = 1$, resultado en **ppb ($\mu\text{g/L}$)**.
- **Lodo / Sedimento (Frasco 0 - Tapa Roja):** $f_{\text{matriz}} = 10$ (extracción: *1 g de lodo + 10 mL Frasco 0, agitar 2 min, decantar 2 min, filtrar*), resultado en **ppb base seca ($\mu\text{g/kg}$)**.

---

## 4. Flujo "Plug & Play" y Código de Tapas

| Etapa | Contenedor | Tapa | Función |
| :--- | :--- | :--- | :--- |
| **Paso 0** | Frasco 0 (solo Lodo) | **Roja** | Extracción 1:10 en base seca (1 g lodo + 10 mL reactivo). |
| **Nivel 1** | Tubo A | **Amarilla** | Tamizaje rápido de $\text{Hg}^{2+}$ inorgánico con AgNPs. Si es **Verde**, finaliza aquí. |
| **Nivel 2a** | Tubo B | **Morada** | Foto-oxidación UV-C 254 nm (se habilita automáticamente si Nivel 1 es Amarillo/Rojo). |
| **Nivel 2b** | Tubo C | **Naranja** | Cuantificación de $\text{Hg}_{\text{total}}$ con AgNPs y cálculo de $\text{CH}_3\text{Hg}^+$. |

---

## 5. Semáforo Normativo (Bolivia e Internacional)

### Matriz Agua ($\mu\text{g/L}$ o ppb)
*Referencias: **Norma Boliviana NB 512** (límite potable: **1 ppb**) y **Ley 1333 — Reglamento en Materia de Contaminación Hídrica (RMCH)** (cuerpos de agua y descargas: **1–2 ppb**).*

| Semáforo | Rango | Dictamen Operativo en Campo |
| :--- | :--- | :--- |
| 🟢 **VERDE** | `< 5 ppb` | Sin contaminación aguda detectada. Para certificación legal NB 512 a 1 ppb, enviar dentro del 5% de auditoría a laboratorio. |
| 🟡 **AMARILLO** | `5 – 20 ppb` | Alerta Ambiental — Supera límite RMCH Bolivia (1–2 ppb). Monitorear fuente. |
| 🔴 **ROJO** | `> 20 ppb` | Peligro Crítico — Descarga severa. Envío obligatorio a laboratorio acreditado (CV-AAS / ICP-MS). |

### Matriz Lodo / Sedimento ($\mu\text{g/kg}$ o ppb base seca)
*Referencias: **CCME Canadá** (Guías de Calidad de Sedimentos de Agua Dulce: **ISQG = 170 ppb**, **PEL efecto probable = 486 ppb**).*

| Semáforo | Rango | Dictamen Operativo en Campo |
| :--- | :--- | :--- |
| 🟢 **VERDE** | `< 50 ppb` | Bajo riesgo en sedimento. |
| 🟡 **AMARILLO** | `50 – 170 ppb` | Alerta en sedimento — Activar Nivel 2 (UV-C) para verificar metilación. |
| 🔴 **ROJO** | `> 170 ppb` | Peligro Crítico en lodo (supera umbral CCME 170 ppb) — Activar Nivel 2 (UV-C) y enviar a laboratorio. |

---

## 6. Escenarios Demo Precargados (Modo Jurado)

Para demostraciones en vivo sin cámara ni reactivos físicos, la app genera viales sintéticos en el Canvas y ejecuta exactamente el mismo algoritmo `getImageData()` sobre la ROI 50×50 px:

1. **Agua Limpia (Río Alto)** `[Agua]`: Tubo A `RGB(234,194,32)` → **~1.3 ppb** → 🟢 Verde (finaliza en Nivel 1).
2. **Efluente Minero (Agua)** `[Agua]`: Tubo A `RGB(235,212,102)` → **~44.4 ppb $\text{Hg}^{2+}$** → 🔴 Rojo (descarga severa, habilita Nivel 2).
3. **Lodo Anóxico (Poza Minera)** `[Lodo 10×]`: Tubo A `RGB(235,204,64)` (**~209.3 ppb $\text{Hg}^{2+}$**) + Tubo C post UV-C `RGB(235,216,120)` (**~554.8 ppb $\text{Hg}_{\text{total}}$** → **~345.5 ppb $\text{CH}_3\text{Hg}^+$**) → 🔴 Rojo (foco activo de metilación).

---

## 7. Estructura del Proyecto y Despliegue en GitHub Pages

```text
.
├── index.html   # PWA estática mobile-first (manifest inline, cámara trasera, Canvas ROI)
├── styles.css   # Diseño limpio, responsivo, código de tapas y semáforo normativo
├── app.js       # Procesamiento real en Canvas, RGB→HSV, calibración, localStorage y CSV
└── README.md    # Documentación técnica, normativa boliviana y referencias
```

### Despliegue rápido en GitHub Pages
1. Sube estos 4 archivos a la rama `main` de tu repositorio en GitHub.
2. Ve a **Settings → Pages → Build and deployment → Source: Deploy from a branch (`main` / `root`)**.
3. Abre la URL HTTPS generada desde tu celular (HTTPS habilita el acceso nativo a la cámara trasera e instalación como PWA).

---

## 8. Referencias Técnicas y Normativas

1. **IBNORCA — Norma Boliviana NB 512:** *Agua Potable — Requisitos* (límite máximo admisible de mercurio total: $0.001\text{ mg/L} = 1\text{ ppb}$).
2. **Estado Plurinacional de Bolivia — Ley N° 1333 de Medio Ambiente:** *Reglamento en Materia de Contaminación Hídrica (RMCH)*, Cuadro A-1 y límites de descarga para cuerpos receptores ($1\text{–}2\text{ ppb}$ de Hg).
3. **CCME (Canadian Council of Ministers of the Environment, 1999):** *Canadian Sediment Quality Guidelines for the Protection of Aquatic Life — Mercury* (ISQG: $170\;\mu\text{g/kg}$; PEL: $486\;\mu\text{g/kg}$).
4. **Convenio de Minamata sobre el Mercurio (PNUMA):** Directrices técnicas para monitoreo de mercurio en sitios contaminados por minería aurífera artesanal (ASGM).
5. **Farhadi, K. et al. (2012):** *Highly selective $\text{Hg}^{2+}$ colorimetric sensor using green synthesized and unmodified silver nanoparticles*, Sensors and Actuators B: Chemical, 161(1), 880–885.
6. **US EPA Method 7473 / Method 1630:** Referencias estándar de laboratorio acreditado para mercurio total (CV-AAS/TDA) y metilmercurio en aguas y sedimentos.
