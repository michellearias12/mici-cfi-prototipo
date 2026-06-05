# Sistema Digital CFI — MICI Panamá
## Certificado de Fomento Industrial — GitHub Pages

Prototipo funcional de alta fidelidad para la digitalización del trámite CFI del Ministerio de Comercio e Industrias de Panamá.

---

## Archivos

| Archivo | Descripción | URL sugerida |
|---|---|---|
| `index.html` | Portal ciudadano — solicitud digital del CFI | `solicitud-cfi.mici.gob.pa` |
| `panel-funcionarios.html` | Panel interno MICI — Secretaria, Analista, CPA, Jefe, Director, Asesor Legal | `funcionarios.cfi.mici.gob.pa` |
| `portal-conapi.html` | Portal CONAPI — agenda de sesión, votación digital, generación de Acta | `conapi.cfi.mici.gob.pa` |

---

## Deployment en GitHub Pages

1. Crear un repositorio en GitHub (ej. `mici-cfi-prototipo`)
2. Subir los tres archivos HTML a la rama `main`
3. Ir a **Settings → Pages → Source → Deploy from branch → main / root**
4. El sitio estará disponible en `https://[usuario].github.io/mici-cfi-prototipo/`

---

## Funcionalidades del prototipo

### Portal ciudadano (`index.html`)
- 9 pasos guiados con barra de progreso interactiva
- Formulario completo con 8 secciones del formulario oficial MICI (Ley 76)
- Selección de tipo de solicitante (persona jurídica / natural)
- Selección de 6 modalidades del CFI
- Carga de los 11 documentos requeridos con confirmación visual
- Tabla de inversiones dinámica con tipos de documento (FF, FC, FE, DA, BP, TB...)
- Cálculo automático del CFI estimado (40%)
- Declaración jurada digital
- Generación de número de expediente y línea de tiempo

### Panel interno MICI (`panel-funcionarios.html`)
- 6 roles de funcionario con cambio desde la barra lateral
- Bandeja filtrada por rol con semáforo de días en trámite
- Secretaria CFI: checklist de 11 documentos + asignación de analista y CPA
- Analista Industrial: agenda de visita técnica + informe técnico con concepto
- Contador (CPA): revisión detalle de inversiones con C/NC por factura
- Jefe de Departamento: revisión y aprobación con historial en línea de tiempo
- Asesor Legal: lista de verificación legal de 6 puntos
- Director DGI: vista previa de Resolución + firma electrónica con PIN
- Vista detallada de cada expediente con línea de tiempo completa

### Portal CONAPI (`portal-conapi.html`)
- 4 vistas navegables: Secretaria, Miembro, Sala de Votación, Generador de Acta
- Confirmación de asistencia de los 6 miembros del Consejo
- Agenda de sesión con 3 expedientes
- Votación electrónica individual por miembro (Favorable / Desfavorable / Abstención)
- Barras de progreso en tiempo real durante la votación
- Resultado automático con concepto oficial
- Generación automática del Acta con datos de la sesión y votos individuales
- Campos de PIN de firma digital por miembro

---

## Tecnologías

- HTML5 + CSS3 + JavaScript vanilla (sin dependencias externas)
- Fuente: DM Sans (Google Fonts)
- Compatible con todos los navegadores modernos
- No requiere servidor — funciona estáticamente en GitHub Pages

---

## Base legal

- Texto Único de la Ley N°76 de 23 de noviembre de 2009
- Decreto Ejecutivo N°37 de 10 de abril de 2018
- Resolución N°12 de 16 de mayo de 2019 (formulario oficial)

---

## Generado para

**Ministerio de Comercio e Industrias (MICI)**  
Dirección General de Industrias  
Departamento de Evaluación Industrial  
República de Panamá
