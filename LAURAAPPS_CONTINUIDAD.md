# LAURAAPPS — MANUAL MAESTRO DE CONTINUIDAD Y RECUPERACIÓN

**Actualizado:** 19 de noviembre de 2026  
**Propósito:** poder mantener, reparar, compilar, firmar, actualizar y republicar las aplicaciones de LauraApps aunque se pierda el chat, cambie el equipo o se interrumpa el acceso a herramientas de IA.  
**Fuente principal:** código y workflows vigentes en GitHub; verificar SIEMPRE los datos actuales en Google Play Console y AdMob antes de publicar.

> ⚠️ Este documento NO contiene contraseñas, claves privadas, archivos JKS, certificados de firma privados, tokens ni códigos de recuperación. Esos elementos deben conservarse por separado, de forma segura. Tener este manual y GitHub NO sustituye tener acceso a la clave de subida y a Google Play Console.

## 1. Accesos y enlaces fundamentales

- **Desarrollador:** LauraApps — cuenta personal de Google Play.
- **Google Play Console:** https://play.google.com/console/u/0/developers/7068660754498827387/home
- **GitHub propietario:** https://github.com/tommymsc
- **AdMob:** https://admob.google.com/
- **Dominio público del desarrollador:** https://tommymsc.github.io/
- **Archivo app-ads.txt:** https://tommymsc.github.io/app-ads.txt
- **Línea requerida en app-ads.txt:** `google.com, pub-2506564165861733, DIRECT, f08c47fec0942fa0`
- **Cuenta de pagos / identificación:** comprobarla directamente en AdMob; NO registrar datos financieros en este archivo.
- **Importante:** el repositorio web `tommymsc/tommymsc.github.io` contiene el archivo app-ads.txt con la línea anterior, pero la disponibilidad pública HTTP y la coincidencia del dominio en la ficha Play deben confirmarse cada vez que AdMob falle.

### Estado conocido al elaborar el manual

| Aplicación | Paquete inmutable de publicación | Repositorio | Estado comunicado en Play |
|---|---|---|---|
| Lazos de Luto | `lazosdeluto.gratis` | https://github.com/tommymsc/Lazos-de-Luto-LauraApps | Producción; versión 2.2.6, código 17 (18 nov 2026) |
| Esquelas y Avisos de Luto | `com.lauraapps.esquelas` | https://github.com/tommymsc/Esquelas-Avisos-Luto-LauraApps | Prueba cerrada |
| Foto Exacta: Comprimir KB | `com.lauraapps.foto_exacta` | https://github.com/tommymsc/foto-exacta-android | Prueba cerrada |
| Horóscopo Diario: Signos | `com.lauraapps.dailyhoroscope` | https://github.com/tommymsc/DailyHoroscope-LauraApps | Prueba cerrada |
| Chat Fácil: Sin Guardar Número | `com.lauraapps.chatfacil` | https://github.com/tommymsc/ChatFacil-LauraApps | Prueba cerrada |

**No incluir en el mantenimiento normal:** `lauraapps.com.radioespanaradiopodcast` (Radio España), cuyo estado informado es «Eliminada por Google». Nunca intentar republicarla o reutilizar su paquete sin resolver la causa oficial en Play.

**Regla absoluta:** consultar **Play Console → aplicación → Prueba y lanzamiento / Producción / Pruebas** para obtener la **última versión realmente distribuida**. Un `README`, un ZIP, una rama, un `versionCode` del repositorio o un AAB viejo no acreditan qué archivo se publicó.

## 2. Primeros 15 minutos al recuperar un proyecto

1. Entrar a Play Console con la cuenta autorizada. Confirmar el estado de la cuenta, avisos, versiones, pistas activas y plazo de verificaciones.
2. Identificar **la aplicación y paquete exacto**. No cambiar `applicationId` para actualizar la misma ficha.
3. Abrir el repositorio correcto y leer `README`, workflows de `.github/workflows/`, Gradle o Flutter y los commits recientes.
4. Abrir **Actions**: comprobar última ejecución exitosa, rama, commit, tipo de artefacto y fecha. Los artefactos pueden caducar; el **código fuente y la firma** importan más que el ZIP del artefacto.
5. Confirmar en **Play Console → Integridad de la app / Firma de apps** cuál es el certificado de **clave de subida** y si Google administra la clave de firma de distribución. Comparar la huella del certificado de la nueva compilación cuando corresponda.
6. Verificar que existen los secretos de firma requeridos en GitHub Actions, **sin revelar sus valores**.
7. Revisar AdMob: aplicación vinculada, IDs correctos, políticas, consentimiento UMP, app-ads.txt y si hay limitación de anuncios.
8. Crear una rama de trabajo y una lista de cambios. No editar la versión en producción a ciegas ni sobrescribir archivos fuente con ZIP anteriores.

## 3. Instrucciones verificadas por aplicación

### A. Lazos de Luto — PRIORIDAD: conservar versión estable

- Repo: https://github.com/tommymsc/Lazos-de-Luto-LauraApps
- Paquete: `lazosdeluto.gratis`
- Última versión comunicada como publicada: **2.2.6, código 17**.
- Workflow: `.github/workflows/android.yml`, «Build Lazos de Luto - 50 diseños».
- Ruta prevista de compilación: Gradle Java 17; `assembleDebug` y `bundleRelease`; fuente con **50 JPG** en `app/src/main/assets/lazos/`.
- **ATENCIÓN ESPECIAL:** el `app/build.gradle` inicial del repositorio declara código 9 / minSdk29 / anuncios 24.7.0, pero el workflow aplica sucesivamente `scripts/upgrade_ui.py`, `scripts/monetization_v22.py`, `scripts/compatibility_api23.py` y `scripts/compatibility_api19.py`. El resultado validado en CI usa **código 17, minSdk19, targetSdk36, Google Mobile Ads 22.6.0 y UMP 2.2.0**. Para una versión **18 o superior** hay que revisar y actualizar de forma coherente **scripts, verificaciones y workflow**. Cambiar únicamente `build.gradle` NO basta.
- Artefacto firmado conocido de código 17: https://github.com/tommymsc/Lazos-de-Luto-LauraApps/actions/runs/44401347683/artifacts/14472960820 (el artefacto podría caducar).
- Firma en Actions: secretos **LAZOS_UPLOAD_KEYSTORE_BASE64, LAZOS_UPLOAD_KEYSTORE_PASSWORD, LAZOS_UPLOAD_KEY_ALIAS, LAZOS_UPLOAD_KEY_PASSWORD, LAZOS_UPLOAD_CERT_PEM_BASE64**.
- Particularidad de seguridad: el workflow solo habilita secretos de firma por una condición explícita en `workflow_dispatch` (`google_upload_key_reset_approved`) o por la coincidencia específica del mensaje de commit `release_signed_api19_version17`. **No replicar ciegamente la condición al actualizar:** revisarla y diseñar un flujo seguro para versiones posteriores, sin asumir que el AAB resultante estará firmado.
- **Riesgo a vigilar:** Google Mobile Ads **22.6.0** es antiguo; su soporte/publicación de anuncios debe reevaluarse antes de una actualización de SDK. Una versión moderna puede elevar el Android mínimo y reducir compatibilidad de dispositivos. Nunca prometer ambas cosas sin probarlas.
- Validaciones previas a actualizar: 50 imágenes reales, guardado, favoritos, WhatsApp, mensajes, consentimiento, anuncios de prueba, firma, compatibilidad de dispositivos, Android Vitals.
- Política: https://tommymsc.github.io/lazos-de-luto-privacy.html

### B. Esquelas y Avisos de Luto

- Repo: https://github.com/tommymsc/Esquelas-Avisos-Luto-LauraApps
- Paquete: `com.lauraapps.esquelas`
- En el **código revisado**: versión **1.9.0 / código 9**, `minSdk29`, `targetSdk36` (NO asumir que coincide con el último AAB subido a Play).
- Workflow `.github/workflows/android.yml`: actualmente construye **APK DEBUG**, pruebas Java, validación JS y lint; **no genera por sí mismo un AAB firmado para Play**.
- Para producción: preparar compilación `gradle :app:bundleRelease`, firmarla con la **clave de subida autorizada de ESTA app**, verificar certificado y ensayar la instalación a través de una pista Play.
- Gradle exige propiedades `ADMOB_APP_ID`, `ADMOB_BANNER_UNIT_ID`, `ADMOB_INTERSTITIAL_UNIT_ID` válidas al compilar release. Configurarlas mediante mecanismo seguro (no en commits ni logs).
- Anuncios: banner en biblioteca; intersticial **al regresar tras al menos 3 guardados y 3 minutos**. Editor, selección de fotos y exportación no deben bloquearse con anuncios.
- Funciones: lazos, esquelas, novenarios, homenajes fotográficos y estados WhatsApp, trabajos locales, PNG.
- Antes de producir: validar dedicatorias sin solapamientos, importación de fotos, exportación PNG, imágenes por tamaño, política, permisos, UMP y pruebas en teléfono físico.

### C. Foto Exacta: Comprimir KB

- Repo: https://github.com/tommymsc/foto-exacta-android
- Paquete: `com.lauraapps.foto_exacta` (confirmar generado por `scripts/configurar_android.py` y AAB).
- Proyecto **Flutter**; la CI **genera el subproyecto Android** al compilar. No editar solo `foto_exacta/` generado: editar `app_source/` y/o `scripts/configurar_android.py`.
- Workflow: `.github/workflows/crear-apk.yml`, «Foto Exacta - APK QA y AAB Play».
- Acción manual `paquete=apk_pruebas`: APK debug con anuncios de prueba.
- Acción manual `paquete=aab_play_admob`: validaciones Flutter, `flutter build appbundle --release`, AAB firmado y IDs reales. Requiere secretos:
  **FE_ADMOB_APP_ID, FE_ADMOB_BANNER_ID, FE_PRIVACY_URL, FE_UPLOAD_KEYSTORE_BASE64, FE_KEYSTORE_PASSWORD, FE_KEY_PASSWORD, FE_KEY_ALIAS**.
- La documentación antigua que dice `aab_firmado` o V3 está desactualizada respecto del workflow V4 **aab_play_admob**. Usar el YAML vigente.
- Función: comprimir a KB exactos / límite máximo, dimensiones, presets, comprobar resultado, procesar localmente; monetización actual del código revisado: banners.
- Antes de actualizar: verificar tamaños de salida, guardar en Descargas, errores con fotos grandes, locales ES/EN si se declaran, permisos, AdMob activo en release, política HTTPS, firma de subida.

### D. Horóscopo Diario: Signos

- Repo: https://github.com/tommymsc/DailyHoroscope-LauraApps
- Paquete: `com.lauraapps.dailyhoroscope`.
- En el **código revisado** `app/build.gradle`: versión **2.10.2 / código 12**, `minSdk26`, `targetSdk36` (verificar lo efectivamente publicado).
- Workflow de AAB firmado: `.github/workflows/play-aab.yml`, «Daily Horoscope - Signed Play AAB» → Run workflow.
- Secretos de firma: **ANDROID_KEYSTORE_BASE64, ANDROID_KEYSTORE_PASSWORD, ANDROID_KEY_ALIAS, ANDROID_KEY_PASSWORD**.
- Validar en CI `gradle :app:bundleRelease` y `jarsigner -verify`; artefacto `DailyHoroscope-Play-AAB-signed`.
- Contiene Google Mobile Ads, UMP y Google Play Billing: banner, nativo, intersticial y Premium. Producto de suscripción `horoscope_premium_monthly`, plan `monthly`.
- **Crítico:** probar compras reales en pista de prueba, restauración, reconocimiento de compras, expiración/cancelación, consentimiento, disponibilidad del producto; no afirmar que la vista demo da acceso Premium real.
- Ojo: `PLAY_AAB_INSTRUCCIONES.md` menciona código 10, que ya no coincide con `app/build.gradle` código 12. Confiar en la rama/artefacto y en Play, no en manuales antiguos.

### E. Chat Fácil: Sin Guardar Número

- Repo: https://github.com/tommymsc/ChatFacil-LauraApps
- Paquete release: `com.lauraapps.chatfacil`; paquete **debug**: `com.lauraapps.chatfacil.debug` (NO se puede usar como actualización de la app de Play).
- Código revisado: versión **1.0.4 / código 5**, `minSdk24`, `targetSdk36`.
- Workflow `.github/workflows/android-debug.yml` «Android Debug APK»: pruebas + APK debug solamente. **NO existe en este flujo un AAB release listo para descargar**; preparar y firmar `gradle :app:bundleRelease` usando la clave de subida vigente.
- Para liberar con publicidad real: `-PenableRealAds=true -PadMobAppId=... -PadMobBannerId=...`; el Gradle bloquea releases si la configuración es inválida. Tratar esos IDs como configuración, no publicarlos si no hace falta.
- Funciones: enlace `wa.me`, selección WhatsApp/Business, prefijos, historial, favoritos, plantillas, ES/EN. Sin permisos de contactos, SMS ni llamadas; no automatizar envíos ni sugerir afiliación oficial a WhatsApp/Meta.
- Política: https://tommymsc.github.io/chat-facil-privacy.html
- Verificar interacción real en móvil y que la navegación externa y los enlaces funcionan.

## 4. Procedimiento estándar para actualizar una app existente

**ANTES DE MODIFICAR**
1. Abrir la ficha Play y anotar paquete, pista, último `versionCode` activo, `versionName`, firma/certificado, países y dispositivos admitidos.
2. Revisar políticas actuales: nivel de API objetivo, SDKs, permisos, publicidad, 16 KB (cuando aplique), declaraciones de datos y requisitos de cuenta.
3. Guardar un **tag Git** del commit exacto de la versión anterior y confirmar que se puede reconstruir; si hay generación de código en CI, guardar el script y la versión de las herramientas.
4. Crear rama `fix/descripcion` o `release/version`; cambiar solo lo necesario.

**COMPILAR Y PROBAR**
5. Mantener el **mismo package/applicationId**; subir `versionCode` a un valor **superior a todos los usados en Play**, incluidas pistas. Cambiar `versionName` de forma comprensible.
6. Construir APK **debug con anuncios de prueba** y hacer pruebas manuales en teléfono físico. Verificar estabilidad en dispositivos compatibles, permisos, pantalla pequeña, offline y regreso desde otras apps.
7. Generar **AAB release firmado** con **la clave de subida correcta de esa app**. Confirmar el resultado del workflow, logs y firma. Un APK debug o AAB unsigned NO sustituyen un AAB firmado de publicación.
8. Verificar ID de paquete, código, manifest, permisos reales, SDKs incorporados, IDs AdMob release, política HTTPS y potenciales cambios en compatibilidad de dispositivos.

**PUBLICAR**
9. Subir el AAB a **prueba interna o cerrada**; revisar errores y cobertura de dispositivos, instalar mediante Play y ejecutar pruebas completas.
10. Crear versión en **Producción** solo después de comprobar requisitos de la cuenta, aprobación/periodo de pruebas si aplica, declaraciones y avisos de políticas. Enviar a revisión.
11. Confirmar publicación **desde Play Console y desde la ficha pública**, no solamente porque Actions esté verde.
12. Durante las siguientes 24–72 h observar errores, ANR, bloqueos, reseñas, solicitudes e impresiones AdMob. Los cambios en avisos de políticas o verificaciones pueden tardar; no asumir que todo es instantáneo.

## 5. Plan de emergencia / problemas frecuentes

### No puedo entrar a Play Console
- Probar https://play.google.com/console/ con la **cuenta autorizada** y un solo perfil de Google.
- Ventana privada, otro navegador, otra red y recarga forzada; **no borrar claves ni eliminar apps**.
- Si aparece error temporal como `514779B4`, conservar captura, hora, URL (sin tokens) y contactar soporte: https://support.google.com/googleplay/android-developer/gethelp
- Verificaciones obligatorias y estado de comerciante deben revisarse cuando se recupere el acceso; una pantalla de error no prueba suspensión.

### Play rechaza el AAB por firma
- No inventar otra clave ni reemplazar archivos JKS al azar.
- Comparar huella del certificado de subida del AAB con la registrada por Play. Usar el keystore original.
- Si se perdió exclusivamente la **clave de subida** y se usa Play App Signing, seguir el procedimiento oficial de **restablecimiento de clave de subida**. Esto **no** es lo mismo que cambiar la clave de firma administrada por Google.
- Sin acceso a consola/propietario/identidad, el repositorio solo no basta para recuperar publicación.

### Play dice que se pierden dispositivos
- Comparar minSdk, ABIs, `uses-feature android:required`, permisos, manifest y librerías nativas respecto a la versión previa.
- En Lazos, modificar conjuntamente los scripts de compatibilidad y sus verificaciones; registrar la lista de dispositivos perdidos de Play antes de aceptar cambios.
- Nunca marcar `required=false` para hardware verdaderamente indispensable.

### Falla una GitHub Action
- Ir a Actions → ejecución fallida → Job → primer paso rojo; conservar mensaje **sin secretos**.
- Verificar rama/commit, dependencias, SDK, JVM, generación de archivos, variables y artefactos.
- Si falta un secreto, corregirlo desde Settings → Secrets and variables → Actions, sin pegar valores en el chat.
- Un workflow debug exitoso NO prueba que un bundle release esté firmado o sea compatible con Play.

### AdMob no monetiza o no verifica `app-ads.txt`
- Confirmar **paquete de Play**, estado público, vinculación de aplicación AdMob, ID app y unidades correctas.
- Confirmar que el sitio web del desarrollador en Play es exactamente el dominio donde está `/app-ads.txt`.
- Abrir públicamente `https://tommymsc.github.io/app-ads.txt`, verificar **HTTP 200**, texto plano y línea exacta `google.com, pub-2506564165861733, DIRECT, f08c47fec0942fa0`.
- En AdMob revisar rastreo, última actualización, tipo de problema y solicitud de nueva comprobación. Un error de `app-ads.txt` NO se arregla alterando el AAB.
- Si hay aviso «Google Play inhabilitado / preferencia del anunciante» después de republicar, verificar estado de Play y esperar la revisión indicada por AdMob (en la comunicación anterior: puede demorar **hasta 7 días**).
- Para medir monetización, usar **impresiones, ingresos, eCPM y tasa de publicación**, por aplicación y por período. No equiparar solicitudes con impresiones.
- Referencia comunicada por el titular: eCPM **US$3.60 de toda la cuenta, últimos 7 días al 19 nov 2026**; es histórico y puede variar. No tomarlo como eCPM garantizado de cada app.

### Play muestra avisos de políticas
- Revisar el aviso **dentro de la app afectada**, versión/pista, fecha, plazo y detalle exacto. El recuento «0 apps con tareas de políticas» es una fotografía de un momento, no inmunidad futura.
- Consultar documentación oficial antes de actualizar SDKs, manifiestos o datos de seguridad.
- En noviembre de 2026 se comunicó **Lazos en Producción con 0 apps en tareas de políticas**; continuar monitoreando.

### Un equipo / colaborador / IA nuevo retoma el proyecto
- Enviar primero **este manual** y enlace al repositorio, **sin** contraseñas.
- Solicitar auditoría de rama principal + workflow + compilación + última versión de Play antes de proponer cambios.
- Dar permiso de acceso al repo con privilegios mínimos y retirarlo al finalizar.
- Validar cada cambio mediante PR, pruebas y código revisable. La IA no debe asumir que posee credenciales de Google Play ni secretos GitHub.

## 6. Copias de seguridad y continuidad fuera del chat

Guardar por lo menos dos copias privadas, separadas, de:
- Archivos **.jks/.keystore** por aplicación y sus alias, junto a contraseñas en un gestor seguro (no en el mismo texto).
- Códigos de recuperación de 2FA y mecanismos de acceso de la cuenta Google y GitHub.
- Un inventario privado de **huellas SHA-256 de certificados de subida**, asociadas a cada app (la huella pública no equivale a la clave).
- Backup Git completo de los cinco repositorios, incluyendo **historial de commits, tags y workflows**. Si hay ZIPs fuente o assets que solo viven fuera de Git, archivarlos también.
- Último **AAB firmado realmente aceptado** por Google Play, SHA-256, versión, fecha y commit de origen, en almacenamiento privado.
- Exportación o capturas de configuraciones de Play y AdMob que no puedan reconstruirse con Git.
- Este `LAURAAPPS_CONTINUIDAD.md` fuera del repositorio (p. ej., copia local y nube privada).

**Verificación trimestral:** comprobar que se puede clonar un repo, ejecutar las pruebas y generar un AAB firmado en entorno controlado, sin publicar ni exponer secretos.

## 7. Pendientes prioritarios

- **ANTES DEL 1 DE FEBRERO DE 2027:** completar la declaración de **estado de comerciante** de la cuenta LauraApps en Play Console. Es un requisito de cuenta, no un simple aviso de una app.
- Verificar que `app-ads.txt` está accesible públicamente, que AdMob marca cada app como verificada y que las apps nuevas estén vinculadas correctamente.
- Confirmar el estado real de producción y período de prueba de **Esquelas, Foto Exacta, Horóscopo y Chat Fácil**.
- Corregir documentos antiguos que muestran versiones/acciones obsoletas; conservar historial técnico, pero no ejecutar instrucciones obsoletas a ciegas.
- Establecer **workflows de release firmada** para Esquelas y Chat Fácil, que actualmente muestran CI de APK debug en la rama inspeccionada.
- Comprobar mantenimiento del SDK publicitario de Lazos y cualquier incidencia de ads.
- Crear una ficha de despliegue por versión en cada repo con: paquete, versión, commit SHA, certificado de subida, nombre de artefacto, pista Play, fecha y resultado.

## 8. Plantilla de ficha de cada nueva publicación

```text
APP:
PAQUETE:
ESTADO ACTUAL EN PLAY:
PISTA:
VERSION NAME:
VERSION CODE:
COMMIT GITHUB:
RAMA/TAG:
WORKFLOW Y EJECUCIÓN:
AAB FIRMADO (ubicación privada / hash SHA-256):
CERTIFICADO DE SUBIDA (huella SHA-256):
PRUEBAS FÍSICAS REALIZADAS:
ADMOB (IDs reales confirmados, NO copiar secretos):
POLÍTICAS / DATA SAFETY VERIFICADAS:
COMPARACIÓN DISPOSITIVOS ADMITIDOS:
FECHA DE ENVÍO A REVISIÓN:
FECHA DE PUBLICACIÓN:
OBSERVACIONES / PLAN DE ROLLBACK:
```

## 9. Mensaje de inicio para cualquier conversación nueva

> «Estoy retomando el portafolio Android LauraApps. Lee primero `LAURAAPPS_CONTINUIDAD.md` del repositorio `tommymsc/tommymsc.github.io`. Los cinco proyectos Android y sus enlaces están ahí. Antes de tocar nada, inspecciona el repositorio específico, sus workflows, el último commit y la versión/código realmente activos en Google Play. Conserva package, firma de subida, IDs reales de AdMob y compatibilidad. No inventes secretos ni afirmes que un AAB está publicado sin evidencia. Propón una rama con pruebas, compila AAB firmado y solo entonces prepara la actualización para Play». 

---

**Límite importante:** este documento recoge la inspección técnica de repositorios y estados que el titular mostró hasta el 19 de noviembre de 2026. No constituye un certificado de que todas las ramas, claves, políticas o compilaciones futuras sean válidas. En un incidente, verificar siempre el estado real antes de actuar.
