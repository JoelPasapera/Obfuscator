<div align="center">

  <!-- Banner Principal -->
  <a href="#opaco">
    <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=10,24,30&height=160&section=header&text=Opaco&fontSize=42&animation=fadeIn&fontAlignY=32&desc=👉%20Ofusca%20tu%20código%20directamente%20en%20el%20navegador&descAlignY=60&descAlign=50" alt="Opaco Banner" width="100%" />
  </a>

  <h3>Browser-first JavaScript obfuscator for complete code folders.</h3>
  <p><b>Local. Verificado. Empaquetado.</b></p>

  <!-- Badges -->
  <img src="https://img.shields.io/badge/JavaScript-ES2022-yellow?style=flat-square" alt="JavaScript ES2022" />
  <img src="https://img.shields.io/badge/HTML5-100%25-orange?style=flat-square" alt="HTML5" />
  <img src="https://img.shields.io/badge/Browser-Local-blue?style=flat-square" alt="Browser Local" />
  <img src="https://img.shields.io/badge/Build%20step-None-brightgreen?style=flat-square" alt="No build step" />

</div>

---

## 📖 Overview

**Opaco** es una herramienta web para ofuscar una carpeta completa de código JavaScript directamente desde el navegador.

Puede procesar archivos **`.js`, `.mjs` y `.cjs`**, además de detectar y ofuscar los bloques `<script>` inline con código JavaScript dentro de archivos **`.html`, `.htm` y `.xhtml`**. Los archivos que no sean de código se conservan y se incluyen en el resultado final sin modificaciones. fileciteturn5file0L228-L245

El flujo está diseñado con una regla importante: **el código ofuscado se verifica antes de formar parte del ZIP final**. La verificación intenta analizar el resultado como JavaScript normal y también como módulo; si una transformación falla o produce código inválido, Opaco conserva el archivo original en lugar de entregar código roto. fileciteturn5file0L63-L109

Todo el procesamiento principal ocurre en el navegador. Los motores necesarios están embebidos en el propio HTML y, cuando es posible, la ofuscación se distribuye entre Web Workers para no cargar todo el trabajo sobre el hilo principal. fileciteturn5file0L112-L189

---

## ✨ Features

### 📁 Procesamiento de carpetas

- Arrastra una carpeta completa a la interfaz o selecciónala desde el explorador de archivos.
- Conserva la estructura y las rutas relativas de los archivos.
- Detecta automáticamente `.js`, `.mjs`, `.cjs`, `.html`, `.htm` y `.xhtml`.
- Copia otros archivos al resultado sin modificarlos. fileciteturn5file0L191-L245

### 🧩 Ofuscación de JavaScript

La configuración base aplica varias transformaciones mediante **JavaScript Obfuscator**, entre ellas:

- Renombrado de identificadores con nombres mezclados (`mangled-shuffled`).
- Ocultamiento de cadenas mediante `stringArray` con codificación **RC4**.
- Rotación, mezcla e indexación desplazada del arreglo de cadenas.
- Wrappers encadenados para las llamadas al arreglo de cadenas.
- División de cadenas en fragmentos.
- Conversión de números a expresiones.
- Simplificación del código transformado.
- Transformación de claves de objetos. fileciteturn5file0L252-L261

### 🛡️ Protección avanzada

Opaco incluye tres capas opcionales que pueden activarse desde **Opciones avanzadas**:

#### Capa anti-desofuscador — empaquetado

Empaqueta el código ofuscado dentro de un cargador que reconstruye el contenido en tiempo de ejecución. La implementación puede usar compresión `deflate`, un alfabeto Base64 personalizado y una transformación XOR antes de reconstruir el código. El resultado termina ejecutándose mediante `eval`, por lo que puede entrar en conflicto con políticas CSP que bloqueen `eval`. fileciteturn5file0L21-L61

#### Enredar la estructura al máximo

Activa **control-flow flattening** con umbral alto y **dead-code injection**, aumentando la dificultad de seguir visualmente el flujo del programa a costa de generar código más pesado y potencialmente más lento. fileciteturn5file0L252-L261

#### Autodefensa contra inspección

Activa `selfDefending` y `debugProtection`. Está pensada para dificultar la inspección y depuración del código publicado, pero también puede complicar el desarrollo y afectar a usuarios legítimos que trabajen con herramientas del navegador. fileciteturn5file0L252-L261

### 🔎 Detección de riesgos

Antes de la entrega, Opaco inspecciona el AST del código y marca patrones que pueden ser problemáticos después del renombrado u otras transformaciones, por ejemplo:

- `eval()` y `Function()` / `new Function()`.
- Lectura de fuente mediante `.toString()`.
- Dependencia de `constructor.name`.
- Uso de `arguments.callee`.
- Sentencias `with`. fileciteturn5file0L70-L96

Los hallazgos se muestran por archivo y línea para que puedas revisar el código antes de publicarlo.

### ✅ Verificación automática

Cada unidad pasa por una comprobación posterior a la ofuscación. Acorn intenta parsear el resultado con `ecmaVersion: "latest"`, primero como script y después como módulo. Si el código no supera esa prueba, el archivo original se conserva. fileciteturn5file0L63-L69

Cuando el empaquetado está activo, Opaco además comprueba que el contenido reconstruido coincida con el código ofuscado antes de utilizar el wrapper. fileciteturn5file0L21-L61

### 📦 Empaquetado ZIP

El resultado final se construye como un archivo `.zip` usando **JSZip 3.10.1** con compresión `DEFLATE` de nivel 6. Los archivos HTML se reconstruyen manteniendo intactos los scripts que no sean JavaScript o que no superen la verificación. fileciteturn2file1L79-L99

### ⚡ Procesamiento paralelo

Cuando hay suficiente trabajo y el navegador admite `Worker`, Opaco crea un pequeño pool de workers para procesar archivos JavaScript en paralelo. El número de workers se adapta a `navigator.hardwareConcurrency`, con un máximo de 8 y limitado por la cantidad de archivos. Si los workers no están disponibles o fallan, se utiliza un fallback en el hilo principal. fileciteturn5file0L127-L189

### 📊 Resultados detallados

Al finalizar, la interfaz presenta:

- Archivos protegidos y verificados.
- Archivos que no pudieron protegerse y cuyo original fue conservado.
- Unidades empaquetadas.
- Archivos copiados sin cambios.
- Comparación visual **Antes / Después** para una muestra de código cuando existe un resultado válido.
- Detalle por archivo y riesgos detectados. fileciteturn2file2L136-L151

---

## 🧠 Cómo funciona

El procesamiento sigue, en esencia, este flujo:

```text
Carpeta del usuario
        │
        ▼
Identificación de archivos
        │
        ├── .js / .mjs / .cjs ─────────────┐
        │                                   │
        ├── HTML + <script> inline ─────────┤
        │                                   ▼
        │                           Escaneo de riesgos
        │                                   │
        │                                   ▼
        │                              Ofuscación
        │                                   │
        │                                   ▼
        │                              Verificación
        │                                   │
        │                      ┌────────────┴────────────┐
        │                      │                         │
        │                    OK                       FALLA
        │                      │                         │
        │                      ▼                         ▼
        │                Opcional pack             Conservar
        │                      │                    original
        │                      ▼                         │
        └──────────────────────┴─────────────────────────┘
                               │
                               ▼
                           ZIP final
```

Para los HTML, Opaco localiza los `<script>` inline que correspondan a JavaScript y reemplaza únicamente aquellos que superan el proceso. Los scripts externos (`src`) y otros tipos como JSON o templates no se modifican desde ese bloque inline. fileciteturn2file1L79-L94

---

## 🔐 Privacy

**Opaco está diseñado para procesar el código localmente en el navegador.**

La herramienta no utiliza etiquetas `<script src="...">` para cargar las dependencias de ejecución: los motores están integrados dentro del propio documento HTML. La arquitectura de workers también construye su fuente desde esos motores embebidos. fileciteturn7file0L1-L4 fileciteturn5file0L112-L125

Esto permite utilizar la herramienta sin subir la carpeta del proyecto a un backend propio de Opaco.

> **Importante:** la privacidad del procesamiento local no significa que la ofuscación sea un mecanismo criptográfico. Una aplicación JavaScript que se entrega al navegador del usuario sigue siendo, en última instancia, código que puede analizarse dinámicamente.

---

## ⚠️ Limitations & Compatibility

La ofuscación no es universalmente transparente. Opaco identifica explícitamente patrones que pueden romperse o cambiar de comportamiento al transformar identificadores. Entre los principales casos de riesgo están `eval`, `Function`, `.toString()` sobre funciones, `constructor.name`, `arguments.callee` y `with`. fileciteturn5file0L70-L96

La capa de empaquetado usa `eval` para reconstruir y ejecutar el código, por lo que una **Content Security Policy (CSP)** estricta puede bloquearla. La propia interfaz recomienda desactivarla cuando la web rechaza `eval` o queda en blanco. fileciteturn1file0L24-L45

Las protecciones más agresivas también pueden aumentar el tamaño, el tiempo de ejecución o dificultar la depuración. Deben probarse en el entorno real antes de publicar la aplicación. fileciteturn1file0L32-L45

---

## 🚀 Roadmap

### Done

- [x] Ofuscación de archivos `.js`, `.mjs` y `.cjs`
- [x] Ofuscación de JavaScript inline dentro de HTML
- [x] Preset de protección fuerte
- [x] Empaquetado opcional con reconstrucción en runtime
- [x] Control-flow flattening opcional
- [x] Dead-code injection opcional
- [x] Self-defending y debug protection opcionales
- [x] Detección de patrones de riesgo
- [x] Verificación sintáctica con Acorn
- [x] Conservación automática del original cuando falla la verificación
- [x] Procesamiento con Web Workers y fallback al hilo principal
- [x] Generación de ZIP
- [x] Reporte detallado por archivo

### Planned

- [ ] Exclusiones por archivo o patrón
- [ ] Configuración avanzada editable por el usuario
- [ ] Presets personalizados
- [ ] Pruebas automáticas de ejecución además de validación sintáctica
- [ ] Registro/exportación del reporte de riesgos
- [ ] Integración con proyectos mediante CLI o API local

> El roadmap anterior separa las capacidades que se observan actualmente en el código de ideas de evolución; las tareas futuras no forman parte de la implementación actual.

---

## 🛠️ Technologies

- **HTML5 + CSS3** — interfaz y lógica de selección de carpetas.
- **JavaScript** — aplicación completa, sin paso de compilación.
- **JavaScript Obfuscator** — motor principal de ofuscación y transformaciones AST. El documento contiene la implementación embebida del motor. fileciteturn0file0L245-L269
- **Acorn 8.18.0** — parser utilizado para verificación sintáctica y análisis de riesgos. fileciteturn6file0L10-L38
- **JSZip 3.10.1** — generación del ZIP final. fileciteturn6file0L84-L94
- **pako** — dependencia utilizada por JSZip para compresión/expansión DEFLATE. fileciteturn6file0L84-L94
- **fflate** — compresión `deflateSync` utilizada por el empaquetado opcional. fileciteturn4file6L283-L295
- **Web Workers + Blob URLs** — procesamiento paralelo y aislamiento del trabajo pesado del hilo principal. fileciteturn5file0L112-L189

No se observa un sistema de build o bundler externo en el archivo entregado: la aplicación y sus motores se encuentran integrados en un único HTML.

---

## 📦 Installation

Opaco está planteado como una aplicación HTML autónoma.

### Opción 1 — Abrir directamente

Descarga o clona el proyecto y abre el archivo:

```text
ofuscador.html
```

### Opción 2 — Servirlo localmente

Para un entorno de desarrollo más predecible, puedes servir la carpeta con cualquier servidor HTTP estático. Por ejemplo, usando Python:

```bash
python -m http.server 8000
```

Después abre:

```text
http://localhost:8000/ofuscador.html
```

No necesitas `npm install`, un bundler ni un proceso de compilación para la implementación entregada.

---

## 🧪 Usage

1. Abre **Opaco** en el navegador.
2. Arrastra una carpeta de proyecto a la zona central o selecciónala manualmente.
3. Revisa la cantidad de archivos detectados.
4. Mantén **Protección fuerte** como configuración base o abre **Opciones avanzadas**.
5. Activa, según tus necesidades, empaquetado, reordenamiento de flujo y autodefensa.
6. Pulsa **Ofuscar**.
7. Revisa el progreso, los riesgos y el detalle por archivo.
8. Descarga la carpeta protegida como `.zip`.
9. Antes de publicar, prueba la aplicación resultante en su entorno real.

---

## 📄 Licensing & Third-Party Notices

APACHE 2.0

---

## 🤝 Contributing

Las contribuciones son bienvenidas.

Antes de enviar cambios que afecten la ofuscación, conviene comprobar al menos:

- que el código ofuscado siga siendo parseable;
- que los archivos HTML mantengan intactos los scripts no transformables;
- que los archivos fallidos conserven correctamente el original;
- que el empaquetado mantenga la integridad del código reconstruido;
- que el comportamiento siga funcionando en navegadores reales.

---

## ⭐ Why Opaco?

La mayoría de herramientas de ofuscación terminan formando parte de una cadena de desarrollo más compleja. **Opaco** concentra el flujo en una interfaz de navegador sencilla:

- 📂 Seleccionas una carpeta completa.
- 🔐 Proteges JavaScript y scripts inline de HTML.
- ✅ Verificas el resultado antes de entregarlo.
- 🧯 Conservas el original cuando una transformación falla.
- ⚡ Aprovechas Web Workers cuando el navegador lo permite.
- 📦 Descargas todo como un único ZIP.

La idea central es sencilla: **ofuscar, comprobar y empaquetar en un mismo flujo local**.

---

<div align="center">

Made with ❤️ using JavaScript

**Opaco — tu código, pero menos legible.**

</div>
