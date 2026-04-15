---
name: aio-tests-csv
version: 1
description: >
  Genera archivos CSV para importación en AIO Tests (Jira). Cubre formato Classic,
  formato BDD, multi-línea, datasets con parámetros, jerarquía de carpetas y
  buenas prácticas para escribir casos de prueba claros, completos y trazables.
agents: [codex, claude-code, main_agent, general_purpose]
triggers:
  - "generar csv para aio tests"
  - "crear casos de prueba para aio"
  - "importar casos en aio tests"
  - "csv test cases jira aio"
  - "generate aio tests csv"
  - "create test cases csv aio"
---

# AIO Tests — CSV Generation Skill

Esta skill guía la generación de archivos `.csv` listos para importar en
**AIO Tests for Jira** (https://www.aiotests.com).

---

## 1. Conceptos clave

| Concepto | Descripción |
|---|---|
| **Classic** | Casos de prueba con pasos numerados (Step Description + Expected Result) |
| **BDD** | Casos en formato Gherkin (Feature / Scenario / Given / When / Then) |
| **Single-line** | Todo el caso en una sola fila del CSV |
| **Multi-line** | Cada paso es una fila; el caso se identifica por `Existing Case ID` repetido |
| **Dataset** | Conjunto de parámetros para casos data-driven; separados por `pipe` (`\|`) |
| **Parámetro en paso** | Variable en un paso encerrada en ángulos: `<nombreParam>` |
| **Jerarquía de carpetas** | Separadas por `->`, ej.: `Auth->Login->2FA` |

---

## 2. Campos soportados

### 2.1 Campos estándar

| Columna CSV | Campo AIO Tests | Obligatorio | Notas |
|---|---|---|---|
| `Existing Case ID` | Existing Case ID | ✅ Sí | Número secuencial (1, 2, 3…). No es el Test Key de Jira. |
| `Title` | Title | ✅ Sí | Título del caso. Claro, imperativo, < 120 caracteres. |
| `Description` | Description | No | Contexto del caso. Markdown básico aceptado. |
| `Status` | Status | No | `Active`, `Draft`, `Deprecated`. Default: `Active`. |
| `Priority` | Priority | No | `Critical`, `High`, `Medium`, `Low`, `Lowest`. |
| `Labels` | Labels | No | Separadas por coma: `smoke,regression,login`. |
| `Components` | Components | No | Componente Jira del proyecto. |
| `Folder` | Folder | No | Jerarquía con `->`. Ej.: `Auth->Login`. |
| `Step Description` | Step Description | ✅ Si hay pasos | Qué hace el tester. Verbo imperativo. |
| `Expected Result` | Expected Result | ✅ Si hay pasos | Resultado observable esperado. |
| `Requirements` | Requirements | No | Jira Issue Keys separados por coma: `PROJ-10,PROJ-11`. |
| `Datasets` | Datasets | No | Pipe-separated. Ver sección 5. |

### 2.2 Campos BDD

| Columna CSV | Campo AIO Tests | Obligatorio |
|---|---|---|
| `Feature` | Feature | ✅ Sí |
| `Scenario` | Scenario / Title | ✅ Sí |
| `Background` | Background | No |
| `Given` | Given step | No |
| `When` | When step | No |
| `Then` | Then step | No |
| `And` | And step | No |
| `Template` | Test Case BDD | ✅ Para BDD |

---

## 3. Formato Classic — Single-line

Cada caso ocupa **una sola fila**. Todos los pasos se concatenan en la celda
de `Step Description` separados por salto de línea interno o numerados en texto.

### Reglas
- La primera fila es el **header**. No omitir.
- `Existing Case ID` debe ser un entero único y secuencial (1, 2, 3…).
- `Step Description` y `Expected Result` son **obligatorios** cuando el caso tiene pasos.
- Si no hay pasos, dejar esas columnas vacías.
- Encoding: **UTF-8**.
- Separador: **coma** (`,`). Si un valor contiene comas, encerrar en comillas dobles.

### Ejemplo mínimo

```csv
Existing Case ID,Title,Priority,Status,Folder,Step Description,Expected Result
1,Verificar login con credenciales válidas,High,Active,Auth->Login,"1. Ingresar usuario 'admin@ejemplo.com'\n2. Ingresar contraseña correcta\n3. Clic en 'Iniciar sesión'","El usuario es redirigido al dashboard con mensaje 'Bienvenido'"
2,Verificar login con contraseña incorrecta,High,Active,Auth->Login,"1. Ingresar usuario válido\n2. Ingresar contraseña incorrecta\n3. Clic en 'Iniciar sesión'","Se muestra error 'Contraseña incorrecta'. El usuario permanece en la pantalla de login."
3,Verificar sesión expira tras inactividad,Medium,Active,Auth->Session,"1. Iniciar sesión\n2. Esperar 30 minutos sin actividad","El sistema cierra la sesión automáticamente y redirige a login con mensaje de expiración."
```

---

## 4. Formato Classic — Multi-line

Cada **paso** ocupa una fila independiente. El caso se identifica porque
`Existing Case ID` y `Title` se **repiten** en cada fila del mismo caso.

### Cuándo preferir multi-line
- Casos con más de 3 pasos.
- Cuando se necesitan expected results individuales por paso.
- Mejor legibilidad y edición en spreadsheets.

### Ejemplo multi-line

```csv
Existing Case ID,Title,Priority,Status,Folder,Step Description,Expected Result,Requirements
1,Crear cuenta de usuario nueva,High,Active,Users->Registration,Navegar a /register,La página de registro se carga correctamente,PROJ-42
1,Crear cuenta de usuario nueva,High,Active,Users->Registration,Completar el formulario con datos válidos,Todos los campos muestran validación verde,PROJ-42
1,Crear cuenta de usuario nueva,High,Active,Users->Registration,Clic en 'Crear cuenta',Se envía email de confirmación y se muestra mensaje de éxito,PROJ-42
2,Intentar registro con email duplicado,High,Active,Users->Registration,Navegar a /register,La página de registro se carga,PROJ-43
2,Intentar registro con email duplicado,High,Active,Users->Registration,Ingresar email ya registrado y completar el formulario,Formulario aceptado visualmente,PROJ-43
2,Intentar registro con email duplicado,High,Active,Users->Registration,Clic en 'Crear cuenta',Aparece error inline: 'Este email ya está en uso',PROJ-43
```

> **Regla de oro multi-line**: todos los campos no-paso (Title, Priority, Status,
> Folder, Requirements) deben ser **idénticos** en cada fila del mismo caso.

---

## 5. Datasets — Casos Data-Driven

Para casos parametrizados, agregar una columna `Datasets` con los valores
separados por `|` (pipe). En los pasos, usar `<nombreParametro>`.

### Estructura del dataset

```
valor1Param1|valor1Param2||valor2Param1|valor2Param2
```

- Un bloque por dataset separado por `||` (doble pipe).
- Dentro de un bloque, cada valor por campo separado por `|`.
- El orden de valores corresponde al orden de parámetros en los pasos.

### Ejemplo con dataset

```csv
Existing Case ID,Title,Priority,Status,Folder,Step Description,Expected Result,Datasets
1,Login con múltiples roles,High,Active,Auth->Roles,"1. Ingresar email '<email>'\n2. Ingresar contraseña '<password>'\n3. Clic en Iniciar sesión","El usuario '<email>' accede con rol '<expectedRole>'","admin@test.com|Admin123|Administrator||editor@test.com|Editor123|Editor||viewer@test.com|Viewer123|Viewer"
```

---

## 6. Formato BDD

Para casos Gherkin, agregar una columna `Template` con valor `Test Case BDD`
y mapearla al campo `Test Case BDD` durante la importación.

### Ejemplo BDD

```csv
Existing Case ID,Template,Feature,Scenario,Given,When,Then,And,Priority,Folder
1,Test Case BDD,Autenticación,Login exitoso con credenciales válidas,el usuario está en la página de login,el usuario ingresa email 'user@test.com' y contraseña 'Pass123',el usuario es redirigido al dashboard,,High,Auth->BDD
2,Test Case BDD,Autenticación,Login fallido con contraseña incorrecta,el usuario está en la página de login,el usuario ingresa email 'user@test.com' y contraseña 'wrongpass',el sistema muestra el mensaje 'Credenciales inválidas',el usuario permanece en la página de login,High,Auth->BDD
3,Test Case BDD,Carrito de compras,Agregar producto al carrito,el usuario está autenticado y en la página del producto,el usuario hace clic en 'Agregar al carrito',el producto aparece en el carrito con cantidad 1,el total del carrito se actualiza correctamente,Medium,Shop->Cart->BDD
```

---

## 7. Buenas prácticas para escribir casos de prueba

### 7.1 Títulos

| ❌ Malo | ✅ Bueno |
|---|---|
| `Test login` | `Verificar login exitoso con credenciales válidas` |
| `Check cart` | `Agregar producto al carrito desde página de detalle` |
| `Error case` | `Mostrar error al intentar login con cuenta bloqueada` |

**Formato recomendado para el título:**
> `[Verbo] [Objeto] [condición/contexto]`
> Ej.: `Verificar descuento aplicado al superar monto mínimo`

### 7.2 Step Description

- Usar **verbos imperativos**: Ingresar, Hacer clic, Navegar, Seleccionar, Verificar.
- Un paso = una acción atómica.
- Incluir datos de prueba específicos: `Ingresar '12345'` no `Ingresar número`.
- Evitar pasos vagos: ❌ `Hacer cosas en el formulario` → ✅ `Completar campo 'Email' con 'test@ejemplo.com'`.

### 7.3 Expected Result

- Describir un **resultado observable y verificable**.
- Incluir: estado del sistema, mensaje mostrado, cambio en UI o datos.
- Evitar resultados vagos: ❌ `Funciona correctamente` → ✅ `Se muestra toast verde 'Cambios guardados'`.

### 7.4 Prioridades

| Prioridad | Cuándo usar |
|---|---|
| `Critical` | Flujos de negocio principales (checkout, login, pago) |
| `High` | Funcionalidades importantes pero no bloquean el core |
| `Medium` | Funcionalidades secundarias o variaciones de flujos principales |
| `Low` | Casos borde, escenarios poco frecuentes |
| `Lowest` | Casos cosméticos o de UX menor |

### 7.5 Etiquetas (Labels) recomendadas

```
smoke         → Suite mínima de sanidad post-deploy
regression    → Regresión completa antes de release
happy-path    → Flujo exitoso principal
negative      → Casos de error o datos inválidos
edge-case     → Límites, valores extremos
integration   → Pruebas entre sistemas/módulos
performance   → Tiempo de respuesta o carga
accessibility → A11y / WCAG
security      → Autenticación, autorización, XSS, etc.
```

### 7.6 Organización de carpetas

```
Feature Principal
└── Sub-feature
    └── Escenario específico

Ejemplos:
  Auth->Login->Credenciales válidas
  Auth->Login->Credenciales inválidas
  Auth->Recuperación de contraseña
  Shop->Carrito->Agregar producto
  Shop->Carrito->Eliminar producto
  Shop->Checkout->Pago exitoso
  Shop->Checkout->Pago rechazado
  API->Usuarios->CRUD
```

---

## 8. Errores comunes y cómo evitarlos

| Error en importación | Causa probable | Solución |
|---|---|---|
| `Story not found [PROJ-XX]` | La Jira issue no existe o pertenece a otro proyecto | Verificar que el issue key sea del mismo proyecto Jira |
| `Cannot create Test Step. Mandatory data is missing` | `Step Description` o `Expected Result` vacío cuando hay pasos | Asegurarse de que ambos campos tengan valor en cada fila de paso |
| Campos mal mapeados | Nombres de columna CSV no coinciden con campos AIO | Usar el field mapping de la UI de importación, o usar los nombres exactos de esta skill |
| Caracteres especiales rompen el CSV | Comas o saltos de línea sin comillas | Encerrar valores complejos en comillas dobles `"..."` |
| Dataset no funciona | Formato de pipe incorrecto | Usar `valor1\|valor2` dentro de comillas. Doble `\|\|` entre datasets |
| Jerarquía de carpetas no creada | Usar `/` en lugar de `->` | Usar exactamente `->` como separador de jerarquía |
| Fechas en campos custom rechazadas | Formato no soportado | Usar formato: `8-Jun-2020` o `Jun 8, 2020` |

---

## 9. Plantilla base — Classic Multi-line (recomendada)

Copiar y adaptar esta plantilla como punto de partida:

```csv
Existing Case ID,Title,Description,Priority,Status,Labels,Folder,Step Description,Expected Result,Requirements
1,[Título del caso],[Descripción opcional del objetivo del caso],High,Active,"smoke,regression",[Feature]->[Sub-feature],[Acción 1],[Resultado esperado de acción 1],[PROJ-XX]
1,[Título del caso],[Descripción opcional del objetivo del caso],High,Active,"smoke,regression",[Feature]->[Sub-feature],[Acción 2],[Resultado esperado de acción 2],[PROJ-XX]
1,[Título del caso],[Descripción opcional del objetivo del caso],High,Active,"smoke,regression",[Feature]->[Sub-feature],[Acción 3],[Resultado esperado de acción 3],[PROJ-XX]
```

---

## 10. Checklist antes de importar

Antes de subir el CSV a AIO Tests, verificar:

- [ ] Primera fila es el header (no datos).
- [ ] `Existing Case ID` es numérico, único por caso, consistente en todas sus filas.
- [ ] `Title` está presente en todas las filas.
- [ ] `Step Description` y `Expected Result` tienen valores en todas las filas que representan pasos.
- [ ] Las celdas con comas o saltos de línea están encerradas en `"..."`.
- [ ] Los `Requirements` usan Jira issue keys válidos del mismo proyecto.
- [ ] Los `Datasets` usan `|` simple dentro de un dataset y `||` entre datasets.
- [ ] Los parámetros en pasos usan `<nombreParam>`.
- [ ] La jerarquía de carpetas usa `->` como separador.
- [ ] El archivo está guardado en **UTF-8**.
- [ ] El archivo no supera **100 MB**.

---

## 11. Referencia rápida — Valores aceptados

```
Status:    Active | Draft | Deprecated
Priority:  Critical | High | Medium | Low | Lowest
Encoding:  UTF-8
Separador: coma (,)
Jerarquía: ->
Datasets:  | entre valores del mismo dataset, || entre datasets
Params:    <nombreParametro> en Step Description
Fechas:    8-Jun-2020 | Jun 8, 2020 | 2020-Jun-8 (entre otros)
Tamaño:    máximo 100 MB
```

---

*Para dudas sobre la herramienta: help@aiotests.com*
*Documentación oficial: https://aiosupport.atlassian.net/wiki/spaces/AioTests*
