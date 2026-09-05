# Casa Yuma · Finanzas — Guía de instalación

Este paquete tiene 4 archivos:

- `index.html` — portal de entrada (login + menú de módulos)
- `pagos.html` — Pagos a Proveedores
- `requisiciones.html` — Requisiciones
- `Codigo.gs` — el backend (Google Apps Script) que los tres archivos usan

Tu archivo de **Auditoría** (el que ya tenías) sigue funcionando exactamente igual, con su propio backend y su propio PIN — no lo toca nada de esto. Solo necesita llamarse `auditoria.html` y vivir en la misma carpeta que los demás para que el botón "Entrar" del portal lo abra directo (si prefieres otro nombre, cambia `AUDITORIA_URL` en `index.html`).

## Por qué Pagos y Requisiciones necesitan un paso extra

Auditoría ya tenía su Google Sheet + Apps Script conectados. Pagos y Requisiciones son módulos nuevos, así que necesitan **su propio Google Sheet + Apps Script** (uno solo para ambos, más el portal). Son ~15 minutos, un solo camino, y no se vuelve a tocar salvo que quieras agregar gente o cambiar reglas.

## Paso 1 — Crea el Google Sheet

Crea una hoja de cálculo nueva llamada, por ejemplo, **"Casa Yuma · Finanzas (backend)"**, con estas 4 pestañas (el nombre de cada pestaña debe ser exacto, y la fila 1 debe tener estos encabezados):

### Pestaña `Usuarios`
| A: ID | B: Nombre | C: PinHash | D: Nivel | E: Activo |
|---|---|---|---|---|

Deja `PinHash` vacío — cada quien crea su PIN la primera vez que entra. Copia estas filas para arrancar (ajusta `Nivel` según corresponda: `read`, `capture` o `full` — **solo quien tenga `full` puede usar Pagos y Requisiciones**):

```
CY035	Aida Carrillo Patiño			read	SI
CY037	Omar Emiliano de la Tejera Zamorano		read	SI
CY038	Rodolfo Ramirez Pérez			read	SI
CY057	Arleth Paola Anicacio Antúnez			read	SI
CY039	Jesus Israel Castillo Hernández			read	SI
CY001	Manuel Arturo Otazo Aponte			full	SI
CY064	Jorge Guillén Garbuno			read	SI
CY070	América Arróniz			read	SI
CY999	Growthbis			read	SI
```
(Deja la columna PinHash vacía al pegar — el ejemplo de arriba la salta.)

Sube de nivel a `full` a quien tú decidas que debe aprobar pagos y requisiciones (por ejemplo, tú mismo y quien te apoye en administración). Esto es solo cambiar una celda, sin tocar código.

### Pestaña `Proveedores`
| A: ID | B: Nombre | C: Categoria | D: Contacto | E: Telefono | F: Email | G: RFC | H: CLABE | I: Notas | J: Activo | K: FechaAlta |
|---|---|---|---|---|---|---|---|---|---|---|

Déjala vacía (solo encabezados) — se llena desde la pestaña "Proveedores" de `pagos.html`.

### Pestaña `Pagos`
| A: ID | B: FechaSolicitud | C: SolicitanteID | D: SolicitanteNombre | E: Proveedor | F: Categoria | G: Concepto | H: Monto | I: Moneda | J: FormaPago | K: FechaRequerida | L: Factura | M: Liga | N: Urgencia | O: Estatus | P: AprobadoPor | Q: FechaAprobacion | R: MotivoRechazo | S: FechaPago | T: ReferenciaPago | U: Notas |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

Déjala vacía (solo encabezados).

### Pestaña `Requisiciones`
| A: ID | B: FechaSolicitud | C: SolicitanteID | D: SolicitanteNombre | E: Departamento | F: Articulo | G: Cantidad | H: Unidad | I: Justificacion | J: Urgencia | K: Presupuesto | L: ProveedorSugerido | M: FechaRequerida | N: Estatus | O: AprobadoPor | P: FechaAprobacion | Q: MotivoRechazo |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

Déjala vacía (solo encabezados).

## Paso 2 — Pega el código del backend

1. En el Sheet: **Extensiones → Apps Script**.
2. Borra el contenido de `Code.gs` que abre por default.
3. Pega ahí el contenido completo de `Codigo.gs` (el archivo que te entregué).
4. Guarda (ícono de disco o Ctrl/Cmd+S).

## Paso 3 — Publica como aplicación web

1. Arriba a la derecha: **Implementar → Nueva implementación**.
2. Tipo: **Aplicación web**.
3. "Ejecutar como": **Yo** (tu cuenta).
4. "Quién tiene acceso": **Cualquier usuario**.
5. Clic en **Implementar**, autoriza los permisos que pida Google (es tu propio script accediendo a tu propio Sheet).
6. Copia la **URL de la aplicación web** que te entrega — termina en `/exec`.

Cada vez que edites `Codigo.gs` después de esto, necesitas volver a **Implementar → Gestionar implementaciones → Editar (lápiz) → Nueva versión → Implementar** para que los cambios se reflejen en la URL publicada (la URL en sí no cambia).

## Paso 4 — Conecta los 3 archivos HTML

Abre `index.html`, `pagos.html` y `requisiciones.html` con cualquier editor de texto y busca esta línea en cada uno (está cerca del inicio del `<script>`):

```js
WEBAPP_URL:'PEGA_AQUI_LA_URL_DE_TU_WEB_APP',
```

Reemplaza el texto entre comillas por la URL que copiaste en el Paso 3, en **los tres archivos**. Guarda.

## Paso 5 — Súbelos a algún lugar accesible por navegador

Para que el portal recuerde tu sesión al pasar de `index.html` a `pagos.html`/`requisiciones.html` (sin pedirte el PIN dos veces), los tres archivos deben vivir en el mismo sitio web — no basta con abrirlos sueltos con doble clic en cada uno.

Opciones sencillas y gratuitas:
- **Netlify Drop** (netlify.com/drop): arrastras la carpeta con los 4 archivos (incluye `auditoria.html`) y te da una URL al instante. Es la opción más rápida.
- **GitHub Pages**: si ya usas GitHub, subes la carpeta a un repositorio y activas Pages.
- Cualquier hosting estático que ya uses para la web de Casa Yuma.

Si prefieres seguir abriendo los archivos con doble clic desde tu computadora (sin subir a ningún lado), todo funciona igual — solo que te pedirá tu PIN otra vez al entrar a Pagos o Requisiciones desde el portal, en vez de recordarlo automáticamente.

## Cómo funciona el acceso (clearance)

- Todos entran con su **número de colaborador + PIN** (el PIN se crea la primera vez, igual que en Auditoría).
- El **Nivel** (`read` / `capture` / `full`) se controla desde la pestaña `Usuarios` del Sheet — es la única fuente de verdad.
- **Pagos** y **Requisiciones** están restringidos a nivel `full`, tanto en el portal (no aparecen como opción) como en el backend (aunque alguien intente entrar directo a `pagos.html`, el servidor rechaza la sesión si no es `full`).
- Auditoría conserva su propio sistema de acceso, sin cambios.

Si más adelante quieres, por ejemplo, que el equipo de Compras pueda **crear** requisiciones sin poder **aprobarlas**, es un ajuste sencillo sobre este mismo backend — dímelo y lo agregamos.

## Ideas para un próximo módulo

El portal (`index.html`) ya tiene un espacio reservado ("Próximamente") para cuando se te ocurra el siguiente: presupuesto anual, nómina, control de activos, lo que sea — se agrega como una tarjeta más sin rehacer nada de lo existente.
