# Dev Journal

Este archivo es la memoria operativa del repo para evitar redescubrir el
proyecto desde cero en cada sesion.

## Instruccion para Codex

Al empezar una sesion de trabajo en este repo, leer primero este archivo y usarlo
como mapa inicial. Despues leer solo los archivos necesarios para la tarea actual.

Cada vez que una tarea cambie arquitectura, flujos, scripts, servicios,
convenciones, validaciones o decisiones de desarrollo, actualizar este archivo en
el mismo cambio. Mantenerlo breve, accionable y enfocado en informacion que ayude
a futuras sesiones.

No usar este journal como reemplazo de `README.md`. Si cambia el uso para el
usuario, actualizar tambien `README.md` o el archivo formal que corresponda.

## Proposito del repo

Configuracion personal ZMK para un Keyball61 Bluetooth. El repo compila firmware
UF2 con GitHub Actions para `nice_nano_v2` y shields `keyball61_left`,
`keyball61_right` y `settings_reset`.

El uso normal es editar configuracion, pushear, descargar el artifact
`firmware.zip` de Actions y flashear el UF2 del lado correspondiente.

## Modelo mental

- `config/keyball61.keymap`: layout, behaviors y layers de teclas.
- `config/boards/shields/keyball61/keyball61.dtsi`: matriz fisica,
  `default_transform` y layout fisico. No tocar para cambios normales de teclas.
- `config/boards/shields/keyball61/keyball61_right.overlay`: hardware del lado
  derecho, trackball PMW3610, OLED y layers del trackball.
- `config/boards/shields/keyball61/keyball61_right.conf`: parametros del
  trackball y ZMK Studio.
- `keymap-drawer/` y `keymap_drawer.config.yaml`: dibujo visual del keymap; no
  cambia el firmware. No regenerarlo localmente; dejar que GitHub Actions lo
  haga.

## Flujo recomendado

Para cambios normales de layout:

```bash
git status --short --branch
editar config/keyball61.keymap
git diff --check
git add .
git commit -m "descripcion corta"
git push
```

Si el push falla porque el remoto avanzo:

```bash
git fetch origin
git rebase origin/main
git push
```

El build real ocurre en GitHub Actions. No compilar localmente y no regenerar
imagenes/dibujos localmente; todo eso debe hacerlo GitHub Actions.

## Archivos clave

- `README.md`: documentacion operativa para cambiar, compilar y flashear.
- `build.yaml`: matriz de firmwares que compila Actions.
- `config/west.yml`: dependencias ZMK y `zmk-pmw3610-driver`.
- `config/keyball61.conf`: configuracion general Bluetooth/display/split.
- `config/keyball61.keymap`: keymap principal; este suele ser el archivo mas
  editado.
- `config/boards/shields/keyball61/keyball61_right.conf`: CPI, scroll, snipe,
  timeout de automouse y ZMK Studio.
- `config/boards/shields/keyball61/keyball61_right.overlay`: `automouse-layer`,
  `scroll-layers`, `snipe-layers`, SPI/I2C y trackball.
- `keymap-drawer/keyball61.yaml` y `.svg`: salida visual/generada del keymap.

## Convenciones importantes

- No modificar los `#define` de layers en `config/keyball61.keymap` sin revisar
  primero la numeracion real de layers en el orden de nodos.
- Para cambios de keymap, normalmente basta flashear el lado derecho; el lado
  derecho es central/master.
- `settings_reset` no es firmware normal de uso diario; sirve para borrar
  settings/pairings.
- Si se edita el keymap, no regenerar `keymap-drawer/` localmente. El workflow
  de GitHub Actions debe producir el dibujo.
- Preferir `git fetch origin` + `git rebase origin/main` cuando GitHub rechace
  el push por remoto adelantado.
- Ignorar siempre `config/boards/shields/keyball61/keyball61_left.conf` y
  `config/boards/shields/keyball61/keyball61.conf` aunque esten vacios; se
  mantienen como placeholders del shield.
- Ignorar la metadata `underglow` en `keyball61.zmk.yml`; viene heredada y no
  implica que haya configuracion RGB activa.
- Ignorar los mappings viejos/no usados de `keymap_drawer.config.yaml` salvo que
  la tarea sea limpiar especificamente la configuracion del drawer.

## Servicios o modulos actuales

- Firmware ZMK split para Keyball61.
- Trackball PMW3610 en el lado derecho.
- OLED SSD1306 en ambos lados.
- ZMK Studio habilitado en el firmware del lado derecho con snippet
  `studio-rpc-usb-uart`.
- Dibujo automatico/manual del keymap con `keymap-drawer`.

## Paths y defaults importantes

- Firmware derecho esperado: `keyball61_right-nice_nano_v2-zmk.uf2`.
- Firmware izquierdo esperado: `keyball61_left-nice_nano_v2-zmk.uf2`.
- Artifact esperado de Actions: `firmware.zip`.
- Vault local de firmwares probados: `vault/`.
- Trackball:
  - `automouse-layer = <4>`
  - `scroll-layers = <5>`
  - `snipe-layers = <6>`
  - `LOCK = 7`, usado para bloquear teclado y trackball al mover el teclado.
  - `TBLOCK = 8`, usado para neutralizar solo el trackball dejando el keymap
    activo.
  - `CONFIG_PMW3610_CPI=1200`
  - `CONFIG_PMW3610_CPI_DIVIDOR=1`
  - `CONFIG_PMW3610_AUTOMOUSE_TIMEOUT_MS=700`

## Secretos y entorno

No hay secretos en el repo. Para pushear a GitHub se usan las credenciales Git
locales del usuario; Codex puede fallar con HTTPS si el entorno no puede pedir
usuario/token.

## Validaciones utiles

```bash
git diff --check
awk '/^  [A-Z]+:/{if(layer){print layer, count}; layer=$1; count=0; next} /^  - /{count++} END{if(layer){print layer, count}}' keymap-drawer/keyball61.yaml
```

La segunda validacion debe mostrar 61 posiciones para cada layer dibujada.
No correr `west build`, `keymap draw`, `keymap parse` ni generadores de imagenes
localmente.

## Estado conocido

- Al 2026-06-03, la validacion completa es GitHub Actions. No intentar instalar
  ni correr toolchains locales para compilar o dibujar.
- `config/boards/shields/keyball61/keyball61_left.conf` y
  `config/boards/shields/keyball61/keyball61.conf` existen pero estan vacios;
  ignorarlos siempre salvo pedido explicito.
- Los `#define` de layers en `config/keyball61.keymap` deben mantenerse
  sincronizados con el orden real de nodos del keymap.

## Cambios recientes relevantes

- `SCROLL` tiene un behavior con hold momentaneo y doble tap para lock; dentro
  de `SCROLL`, la misma tecla apaga el lock. El doble tap de scroll usa
  `tapping-term-ms = <350>` para permitir una pulsacion menos rapida.
- Los pulgares izquierdos principales y un pulgar derecho fueron simplificados
  a `SPACE`.
- La tecla mas a la izquierda del pulgar derecho replica el behavior de la tecla
  de scroll: hold momentaneo a `SCROLL`, doble tap para lock.
- En `SCROLL`, `[` activa `FUN` momentaneamente y `]` activa `SYM`
  momentaneamente; `G` es `PgUp`, `B` es `PgDn`.
- En `SCROLL`, las posiciones que en default son `SPACE` pasan a ser `ENTER`.
- En `SCROLL`, `Z` dispara una macro que escribe
  `rodrigo.m.rivero13@gmail.com`.
- `LOCK` se activa/desactiva desde `SCROLL` presionando juntas las posiciones
  thumb 54, 55 y 58. La layer usa `&none` en todas las teclas para no caer a
  layers inferiores.
- En `LOCK` y `TBLOCK`, `keyball61_right.overlay` usa una `ball action` del
  driver PMW3610 con cuatro bindings `&none`; como son layers altas, esto
  neutraliza movimiento, scroll y automouse del trackball mientras esten activas.
- La tecla scroll izquierda mantiene doble tap para toggle de `SCROLL`; la tecla
  scroll derecha mantiene hold a `SCROLL`, pero doble tap hace toggle de `MOUSE`.
  En `MOUSE`, ambas teclas scroll hacen hold a `SCROLL` y doble tap apaga
  `MOUSE`.
- `MOUSE` no tiene atajos multimedia en la fila superior; en esa layer `A/S/D`
  son click derecho/medio/izquierdo.
- `FUN` tiene los atajos multimedia en la fila superior, en las mismas posiciones
  donde antes estaban en `MOUSE`.
- En `SCROLL`, las posiciones `6/7/8/9` son `+/*/-//`.
- El umbral de movimiento accidental del PMW3610 esta en
  `CONFIG_PMW3610_MOVEMENT_THRESHOLD=5`.
- El automouse se desactiva rapido:
  `CONFIG_PMW3610_AUTOMOUSE_TIMEOUT_MS=400`.
