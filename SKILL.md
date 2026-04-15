---
name: aio-tests-csv
version: 2
description: >
  Genera, adapta y valida archivos CSV/Excel para importar casos en AIO Tests (Jira).
  Usa esta skill siempre que el usuario quiera crear, convertir, revisar o corregir
  archivos de importación para AIO Tests, especialmente si menciona CSV, Excel,
  single-line, multi-line, Jira Story ID, Test Key, Test Id, datasets, folder
  hierarchy o los formatos oficiales de AIO.
agents: [codex, claude-code, main_agent, general_purpose]
triggers:
  - "generar csv para aio tests"
  - "crear casos de prueba para aio"
  - "importar casos en aio tests"
  - "csv test cases jira aio"
  - "generate aio tests csv"
  - "create test cases csv aio"
---

# AIO Tests — CSV / Excel Import Skill

Esta skill guía la generación y revisión de archivos listos para importar en
**AIO Tests for Jira** usando como **fuente de verdad** los CSV de ejemplo y la
**documentación oficial** de AIO Tests.

## 1. Principios base tomados de la documentación oficial

- La **primera fila** del archivo es siempre el header.
- Los archivos sample oficiales son **ejemplos de conveniencia**, no un único
  esquema obligatorio. La importación real se resuelve con **Field Mapping** y
  **Data Mapping**.
- Siempre mapear una **columna identificadora del caso** (por ejemplo `Test Id`
  o `Test Key`) y una **columna de título** (`Summary` o `Title`) a los campos
  correspondientes de AIO Tests.
- Si el proyecto tiene más campos marcados como obligatorios en su configuración,
  también deben venir informados o mapearse correctamente.
- Si el caso tiene pasos, cada step necesita **texto de paso** y **expected result**.
- La jerarquía de carpetas usa el delimitador `->`.
- Los casos data-driven usan una columna aparte para datasets con `|` y parámetros
  en pasos con `<nombreParametro>`.
- Columnas de usuario como `Creator` pueden requerir **User Mapping**.
- Los CSV oficiales pueden venir de Excel con codificaciones como **Windows-1252**.
  Para archivos nuevos, preferir **UTF-8** si es posible.
- En pantallas/documentación de AIO puede aparecer el nombre del campo destino
  como `Existing Case ID`, aunque los archivos sample usen headers fuente como
  `Test Id` o `Test Key`.

## 2. Esquemas oficiales de ejemplo

### 2.1 Sample clásico oficial para single-line y multi-line

Los archivos sample oficiales de casos clásicos usan este esquema fuente:

- `Test Id`
- `Summary`
- `Priority`
- `TestSteps`
- `ExpectedResults`
- `Story`
- `Test Type`
- `Component`
- `Release`
- `Status`
- `Creator`

Cómo interpretar estas columnas:

- `Summary` es la columna fuente del título del caso.
- `Story` es la columna fuente para vincular requerimientos / historias.
- `TestSteps` y `ExpectedResults` son las columnas fuente para pasos y resultados esperados.
- Los valores de prioridad/estado/tipo en el sample son **valores fuente**, no
  necesariamente los valores internos finales de AIO. El mapeo se hace en la pantalla
  de **Data Mapping**.

### 2.2 Blank template oficial

Uno de los templates oficiales en blanco usa este header fuente:

- `Test Key`
- `Title`
- `Preconditions`
- `Priority`
- `Test Steps`
- `Data for Steps`
- `Expected Results`
- `Jira Story ID`
- `Test Type`
- `Component`
- `Release`
- `Test Case Status`
- `Creator`

Nota importante:

- El sample oficial en blanco actualmente puede incluir una **columna vacía extra**
  antes de `Creator`, por ejemplo:
  `...,Test Case Status,,Creator`
- Si el usuario quiere replicar el archivo oficial **tal cual**, preservar esa columna.
- Si el usuario quiere un template limpio pero compatible, se puede omitir la columna
  vacía siempre que no se necesite mapear nada contra ella.

### 2.3 Consecuencia práctica al generar archivos

- Por defecto, **preferir uno de los esquemas oficiales** anteriores.
- Si el usuario ya tiene una plantilla corporativa existente, **preservar sus headers**
  y adaptar el contenido a ese formato.
- No insistir en headers custom como `Existing Case ID`, `Step Description`,
  `Expected Result`, `Requirements`, etc. como si fueran la única forma válida.
- Si por alguna razón se usan headers custom, avisar que **Field Mapping manual** será necesario.

## 3. Patrón single-line alineado al sample oficial

En el sample single-line oficial, **cada caso ocupa una sola fila** y la columna
`TestSteps` contiene saltos de línea internos dentro de una sola celda.

Reglas:

- Una fila por caso.
- `TestSteps` puede contener varias acciones separadas por saltos de línea internos.
- Celdas con comas o saltos de línea deben ir entre comillas dobles.
- `ExpectedResults` puede resumir el resultado esperado global del caso, siguiendo
  el estilo del sample oficial.
- `Story` o `Jira Story ID` pueden usarse como fuente para trazabilidad con requisitos.

### Template single-line estilo oficial

```csv
Test Id,Summary,Priority,TestSteps,ExpectedResults,Story,Test Type,Component,Release,Status,Creator
1,User should enter valid Pin to get access to his account.,High,"Insert valid card in the insertion point of ATM
Enter the valid pin number
Click on money withdrawal",ATM should display a meaningful insufficient balance message,ADP-1,Manual,,,P,Noopur
```

## 4. Patrón multi-line alineado al sample oficial

En el sample multi-line oficial, la **primera fila del caso** lleva los datos del caso,
 y las **filas de continuación** dejan vacías las columnas de metadatos y sólo añaden
 más pares `TestSteps` + `ExpectedResults`.

Reglas:

- La primera fila del caso contiene `Test Id` y metadatos principales.
- Las filas siguientes del mismo caso pueden dejar vacías las columnas de metadatos.
- Cada fila de continuación representa un paso adicional.
- No exigir que `Summary`, `Priority`, etc. se repitan en cada fila: el sample oficial no lo hace.
- Cada fila que represente un paso debe seguir teniendo **paso + expected result**.
- Si el equipo del usuario ya usa una convención con metadatos repetidos en cada fila,
  se puede respetar, pero el estilo oficial de referencia es el de **continuaciones en blanco**.

### Template multi-line estilo oficial

```csv
Test Id,Summary,Priority,TestSteps,ExpectedResults,Story,Test Type,Component,Release,Status,Creator
1,User should enter valid Pin to get access to his account.,High,Insert valid card in the insertion point of ATM,"ATM Machine should display language page with options such as ENGLISH and SPANISH",ADP-1,Manual,,,P,Noopur
,,,Enter the valid pin number,ATM should display the account type selection page,,,,,,
,,,Click on money withdrawal,ATM should display amount entry page.,,,,,,
,,,Enter an amount greater than balance and click on OK,ATM should display a meaningful insufficient balance message.,,,,,,
```

## 5. Field Mapping y Data Mapping son de primera clase

La skill debe tratar el CSV como **esquema fuente** y a la pantalla de importación de
AIO como la **capa canónica de mapeo**.

Mapeos habituales desde los samples oficiales:

- `Test Id` o `Test Key` -> campo identificador del caso durante la importación
- `Summary` o `Title` -> `Title`
- `TestSteps` o `Test Steps` -> campo de pasos
- `ExpectedResults` o `Expected Results` -> campo de expected result
- `Story` o `Jira Story ID` -> vínculo con historia/requerimiento
- `Status` o `Test Case Status` -> estado del caso
- `Creator` -> campo de usuario si aplica

Los valores del sample oficial deben tratarse como **valores fuente**:

- Prioridades como `High`, `Med`, `Highest`
- Estados como `P`, `F`, `NR`
- Tipos como `Manual`, `Unit`

No normalizar estos valores por defecto. En su lugar:

- generar el CSV siguiendo el estilo oficial si ese es el objetivo,
- y dejar que **Data Mapping** traduzca valores como `Highest` -> `Critical`,
  o `P` -> el estado equivalente del proyecto.

Si AIO auto-mapea algo mal, indicar al usuario que puede usar:

- `Reset Mapping` para mapear desde cero
- `Re-Initialize Mapping` para volver al auto-mapeo de AIO

## 6. Columnas opcionales soportadas por la documentación

Además de los headers que aparecen en los sample files, la documentación oficial deja
claro que se pueden usar columnas adicionales siempre que se mapeen correctamente.

### 6.1 Folder hierarchy

Si el usuario necesita importar casos dentro de carpetas, agregar una columna como `Folder`
 y usar siempre `->` como separador.

Ejemplos:

- `Auth->Login`
- `Auth->Login->2FA`
- `Shop->Checkout->Rejected payment`

### 6.2 Casos data-driven / datasets

Para casos parametrizados:

- agregar una columna separada para datasets,
- mapear esa columna al campo `Datasets` en AIO,
- usar `|` entre valores del mismo dataset,
- usar `||` entre datasets,
- y usar `<nombreParametro>` dentro de los pasos.

Ejemplo:

```text
admin@test.com|Admin123||editor@test.com|Editor123
```

Ejemplo de parámetro en paso:

```text
Enter username <email>
```

### 6.3 Preconditions, Data for Steps y custom fields

- Si el usuario quiere una plantilla estilo blank template, usar `Preconditions` y
  `Data for Steps` cuando aplique.
- Si el proyecto tiene **custom fields obligatorios**, incluirlos en el archivo fuente.
- Para fechas en custom fields, respetar formatos admitidos por AIO como:
  `8-Jun-2020`, `Jun 8, 2020`, `2020-Jun-8`.

## 7. BDD / Gherkin guidance

AIO Tests soporta imports BDD/Gherkin mediante **BDD Excel** y también mediante
**feature files**. Como los sample CSV proporcionados oficialmente en este contexto son
**samples clásicos**, no imponer un esquema CSV BDD inventado como si fuera universal.

Cuando el usuario pida BDD:

1. aclarar si quiere:
   - CSV clásico,
   - BDD Excel,
   - o `.feature` files
2. preferir la plantilla BDD existente del proyecto si ya existe
3. preservar `Scenario Outline` / ejemplos como datasets o examples
4. usar tags / requirement mapping cuando el flujo del proyecto lo necesite

En ausencia de una plantilla BDD real del usuario, ser explícito sobre esta limitación
en vez de inventar headers rígidos.

## 8. Encoding, higiene CSV y seguridad de importación

Antes de dar por terminado un archivo:

- confirmar que la primera fila sea el header
- mantener el archivo por debajo de **100 MB**
- encerrar entre comillas celdas con comas o saltos de línea
- preferir **UTF-8** para archivos nuevos
- aceptar que samples oficiales pueden venir en **cp1252 / Excel encoding**
- no romper smart quotes ni caracteres especiales al re-guardar el archivo
- usar coma `,` como separador salvo que el usuario pida explícitamente otra estrategia
  por un problema local de Excel

## 9. Errores comunes y cómo interpretarlos

- `Story not found [Jira issue key]`
  - El issue no existe, pertenece a otro proyecto o el valor fuente no está bien mapeado.

- `Cannot create Test Step. Mandatory data is missing for the Test Step. Please provide the value for Step and Expected Result`
  - Falta paso o expected result en alguna fila que representa un step.

- Mapeo incorrecto de columnas
  - Los headers fuente existen, pero quedaron mal asignados en Field Mapping.

- Mapeo incorrecto de valores
  - Los valores fuente (`Highest`, `P`, etc.) no se tradujeron bien en Data Mapping.

- Jerarquía de carpetas incorrecta
  - Se usó `/` u otro separador en vez de `->`.

- Error en fechas custom
  - El valor no sigue uno de los formatos soportados por AIO.

## 10. Cómo debe comportarse el agente al usar esta skill

- Tomar como referencia **primaria** los sample files oficiales.
- Elegir por defecto el estilo más cercano al pedido del usuario:
  - `single-line` -> sample clásico con `Test Id` / `Summary`
  - `multi-line` -> sample clásico multi-line con filas de continuación en blanco
  - `blank template` -> sample con `Test Key` / `Title`
- Si el usuario aporta una plantilla interna existente, **preservarla**.
- Explicar cuándo hará falta **Field Mapping** o **Data Mapping** manual.
- No presentar headers custom normalizados como si fueran el único estándar válido.
- Si faltan datos del proyecto (custom fields obligatorios, valores válidos, etc.),
  pedir sólo lo estrictamente necesario.
- Priorizar salidas **listas para importar** y lo más cercanas posible al estilo oficial.

## 11. Templates listos para usar

### 11.1 Single-line estilo sample oficial

```csv
Test Id,Summary,Priority,TestSteps,ExpectedResults,Story,Test Type,Component,Release,Status,Creator
1,User should enter valid Pin to get access to his account.,High,"Insert valid card in the insertion point of ATM
Enter the valid pin number
Enter an amount greater than balance",ATM should display a meaningful error message for insufficient balance,ADP-1,Manual,,,P,Noopur
```

### 11.2 Multi-line estilo sample oficial

```csv
Test Id,Summary,Priority,TestSteps,ExpectedResults,Story,Test Type,Component,Release,Status,Creator
1,User should enter valid Pin to get access to his account.,High,Insert valid card in the insertion point of ATM,"ATM Machine should display language page with options such as ENGLISH and SPANISH",ADP-1,Manual,,,P,Noopur
,,,Enter the valid pin number,ATM should display the account type selection page,,,,,,
,,,Click on money withdrawal,ATM should display amount entry page.,,,,,,
,,,Enter an amount greater than balance and click on OK,ATM should display a meaningful insufficient balance message.,,,,,,
```

### 11.3 Blank template estilo sample oficial

```csv
Test Key,Title,Preconditions,Priority,Test Steps,Data for Steps,Expected Results,Jira Story ID,Test Type,Component,Release,Test Case Status,,Creator
```

## 12. Checklist breve antes de entregar el CSV

- [ ] ¿El esquema elegido coincide con uno de los samples oficiales o con la plantilla del usuario?
- [ ] ¿La primera fila es header?
- [ ] ¿La columna identificadora y la columna de título están presentes?
- [ ] ¿Cada step tiene expected result?
- [ ] ¿Las celdas con saltos de línea/comas están entre comillas?
- [ ] ¿Se explicó si hará falta Field Mapping o Data Mapping?
- [ ] ¿Los datasets usan `|` / `||` y parámetros `<param>` si aplica?
- [ ] ¿La jerarquía usa `->` si hay folders?
- [ ] ¿El archivo final está listo para importar sin sorpresas?

*Soporte: help@aiotests.com*
*Documentación oficial: https://aiosupport.atlassian.net/wiki/spaces/AioTests*
