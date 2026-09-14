# Team Vikingo Rankings (v2 — Login, roles y ranking automático)

SPA de ranking y perfiles de BJJ, sin backend. Incluye:

- `index.html` — interfaz, CSS y lógica completa de la aplicación.
- `datos.json` — base de datos inicial con 6 peleadores de ejemplo (Carlos Alexander Arias Santos incluido).

## Cómo ver la página

`index.html` hace `fetch('datos.json')`, lo cual los navegadores bloquean si abres el archivo con doble clic (protocolo `file://`). Usa un servidor local:

```bash
python3 -m http.server 8000
# abre http://localhost:8000
```

Al subirlo a cualquier hosting estático (GitHub Pages, Netlify, Vercel, etc.) funciona igual, sin configuración adicional.

## Acceso y roles

La página se abre siempre sobre una pantalla de login. No hay forma de ver el ranking sin autenticarse.

| Usuario | Contraseña | Rol | Puede |
|---|---|---|---|
| `Team Vikingo` | `pasandoguardia` | Lector | Ver rankings y perfiles completos. Sin botones de edición. |
| `admin` | `admin_pasandoguardia` | Administrador | Todo lo del lector + añadir/editar/eliminar peleadores, subir fotos y exportar `datos.json`. |

Las credenciales incorrectas muestran "Acceso Denegado" en rojo. La sesión se guarda en `sessionStorage` (dura mientras la pestaña esté abierta); "Cerrar sesión" en el header vuelve al login.

Ambas cadenas de usuario/contraseña y la lógica de roles están en la constante `CREDENTIALS`, al inicio del `<script>` — cámbialas ahí cuando quieras.

### ⚠️ Nota honesta sobre la seguridad

Este login corre **enteramente en el navegador**: no hay servidor que valide nada. Cualquier persona con conocimientos básicos puede abrir "Ver código fuente" y leer las contraseñas directamente en el HTML/JS. Esto sirve para:
- Evitar que un buscador indexe la página (`<meta name="robots" content="noindex, nofollow">`).
- Filtrar el acceso casual de alguien que llegue al link sin permiso.

**No es** seguridad real de nivel producción. Si necesitas proteger datos sensibles de verdad, hace falta autenticación con un backend (por ejemplo, un pequeño servicio con sesiones o un proveedor como Auth0/Firebase Auth), no solo JavaScript en el cliente.

## Vista de Ranking (minimalista, sin fotos)

Cada una de las 3 pestañas (Top General, Cinturones Blancos, Cinturones Azules) muestra una **lista tipográfica limpia**, sin tarjetas ni fotos:

- Número de posición.
- Flecha de tendencia (▲ verde / ▼ roja / — gris), editable por el admin como campo visual.
- Nombre + apodo + punto de color del cinturón.
- Récord (V-D-E) y puntos de ranking.
- La fila #1 se resalta solo con tipografía/color (sin imagen).
- Click en cualquier fila abre el perfil del peleador.

## Ordenamiento 100% automático — CRÍTICO

**No existen posiciones manuales.** Cada peleador tiene un campo `puntosRanking` en `datos.json`. Antes de pintar cualquier lista, el JS hace:

```js
list.sort((a, b) => (Number(b.puntosRanking) || 0) - (Number(a.puntosRanking) || 0));
```

Así que cuando el Admin cambia los puntos de alguien desde el formulario, su posición en la lista (y en el ranking de su cinturón) se recalcula sola al guardar. Lo mismo aplica al añadir o eliminar peleadores.

## Vista de Perfil ("Fighter Card")

Al entrar a un perfil sí se ve la tarjeta grande estilo UFC:

- Foto banner grande (o silueta placeholder si no hay foto), nombre, apodo, cinturón, peso, edad, récord y puntos de ranking.
- **Perfil táctico** (reemplaza a las técnicas favoritas por seguridad): Estilo de Grappling, Años en el Tatami, Frase de Guerra/Lema.
- **Estadísticas de combate**: Victorias por Sumisión, Victorias por Puntos, Efectividad de Pases (%), Defensa General (%).
- En modo Admin aparecen los botones "Editar Perfil" y "Eliminar" arriba a la derecha del banner.

## Herramientas de Admin

- **+ Añadir Peleador** / **Editar Perfil**: formulario completo (datos generales, foto, perfil táctico, estadísticas, puntos de ranking y tendencia visual).
- **Foto**: `<input type="file">` que convierte la imagen a Base64 y la guarda dentro del objeto del peleador.
- **Generar y Descargar datos.json**: descarga el estado actual (con fotos incluidas) para reemplazar el `datos.json` del proyecto y publicar los cambios a todos los visitantes.
- Cada cambio también se respalda en `localStorage` (`teamVikingo_datos_v2`) para no perder trabajo si recargas antes de exportar. Solo el `datos.json` exportado y subido al hosting actualiza la página para todos los demás.

Para reiniciar desde el `datos.json` original del proyecto, borra el respaldo local desde la consola del navegador:

```js
localStorage.removeItem('teamVikingo_datos_v2')
```

## Estructura de `datos.json`

```json
{
  "equipo": "Team Vikingo",
  "actualizado": "2026-09-13",
  "peleadores": [
    {
      "id": "fighter-001",
      "nombre": "Carlos Alexander Arias Santos",
      "apodo": "El Vikingo",
      "cinturon": "Azul",
      "peso": 82,
      "edad": 27,
      "victorias": 18,
      "derrotas": 4,
      "empates": 1,
      "foto": "",
      "puntosRanking": 980,
      "tendencia": "same",
      "estiloGrappling": "Híbrido",
      "aniosTatami": 9,
      "fraseGuerra": "Se gana o se aprende, nunca se pierde.",
      "stats": {
        "victoriasSumision": 11,
        "victoriasPuntos": 7,
        "efectividadPases": 74,
        "defensaGeneral": 88
      }
    }
  ]
}
```

- `cinturon`: `Blanca`, `Azul`, `Violeta`, `Marrón`, `Negra`. Los rankings por cinturón solo existen para `Blanca` y `Azul`; el resto solo aparece en el Top General.
- `puntosRanking`: el único dato que determina la posición. Más alto = más arriba.
- `tendencia`: `"up"`, `"down"` o `"same"` — solo decorativo, no afecta el orden.
- `foto`: `""` para usar el avatar-silueta automático, o `data:image/...;base64,...`.
