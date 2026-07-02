# GAR & RAN Agencias — Plataforma Web (PWA)

Aplicación web progresiva para la gestión de cotizaciones, clientes, contratos, catálogo de unidades y transporte ejecutivo de **GAR & RAN Agencias**.

Construida con **Next.js 14 (App Router) + TypeScript + Tailwind CSS**. Sin backend, sin Firebase y sin Google Maps: todos los datos (cotizaciones, clientes, contratos, unidades, documentos, firmas) se guardan localmente en el navegador con **IndexedDB**. No requiere cuentas ni variables de entorno — se instala y se usa de inmediato.

---

## 1. Cómo funciona el almacenamiento (importante)

Esta versión **no tiene servidor ni base de datos en la nube**. Toda la información se guarda en el propio navegador del dispositivo donde se usa, mediante IndexedDB (a través de la librería [Dexie.js](https://dexie.org)):

- Funciona sin conexión a internet una vez cargada.
- No requiere crear cuentas, proyectos de Firebase ni API keys.
- **Los datos no se sincronizan entre dispositivos ni navegadores.** Si usas la app en el celular y en la computadora, cada uno tendrá su propia información.
- Si el usuario borra los datos de navegación / caché del sitio, o desinstala la PWA, **se pierde la información guardada**. Conviene generar y guardar los PDF de cotizaciones y contratos como respaldo, y hacer descargas periódicas de lo importante.
- No hay inicio de sesión: cualquier persona con acceso al dispositivo puede usar la app.

Si en el futuro quieres sincronizar datos entre dispositivos o tener respaldo en la nube, se puede agregar un backend más adelante (Firebase, Supabase, un servidor propio, etc.) — la capa de datos está aislada en `src/lib/services/` y `src/lib/db.ts` para facilitar ese cambio.

---

## 2. Requisitos previos

- Node.js 18 o superior
- Una cuenta de [GitHub](https://github.com/) y de [Vercel](https://vercel.com/)

No se necesita ninguna otra cuenta ni configuración.

---

## 3. Ejecutar en local (Visual Studio Code)

```bash
npm install
npm run dev
```

Abre [http://localhost:3000](http://localhost:3000) — la app carga directo en el Dashboard.

> Si `npm install` falla por conflictos de dependencias (`ERESOLVE`), ejecuta `npm install --legacy-peer-deps`. El `package.json` ya incluye un `overrides` para evitar este problema en la mayoría de los casos.

---

## 4. Subir el proyecto a GitHub

```bash
git init
git add .
git commit -m "GAR & RAN Agencias - versión local (sin Firebase, sin Google Maps)"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/garran-app.git
git push -u origin main
```

---

## 5. Desplegar en Vercel

1. Entra a [vercel.com/new](https://vercel.com/new) e importa el repositorio de GitHub.
2. Framework Preset: **Next.js** (se detecta automáticamente).
3. No hay variables de entorno que configurar.
4. Haz clic en **Deploy**.
5. Cuando termine, obtendrás una URL pública (por ejemplo `https://garran-app.vercel.app`).

---

## 6. Instalar como app (PWA) sin App Store

**Android (Chrome):** abre la URL de Vercel → menú (⋮) → "Instalar aplicación" / "Agregar a pantalla de inicio".

**iPhone (Safari):** abre la URL de Vercel → botón compartir (□↑) → "Agregar a pantalla de inicio".

La app se instalará con ícono propio y funcionará en modo standalone (pantalla completa, sin barra de navegador). Recuerda que los datos capturados en cada instalación viven solo en ese dispositivo/navegador.

---

## 7. Estructura del proyecto

```
src/
  app/                     Rutas (App Router de Next.js)
    (protected)/           Layout principal con menú lateral / inferior
      dashboard/
      cotizaciones/
      clientes/
      contratos/
      unidades/
      configuracion/
  components/
    ui/                    Sistema de diseño reutilizable
    layout/                Sidebar, BottomNav, Header, AppShell
    quotes/ clients/ units/ contracts/ signature/
  lib/
    db.ts                   Definición de la base de datos local (Dexie / IndexedDB)
    types.ts                 Tipos de datos compartidos
    quote-calc.ts            Lógica de cálculo de cotizaciones
    blob.ts                  Utilidades para manejar imágenes/archivos (Blob)
    services/                 Capa de acceso a datos (CRUD sobre IndexedDB)
    pdf/                      Generación de PDFs (jsPDF)
  hooks/
    useObjectUrl.ts           Convierte un Blob guardado en una URL para <img>
public/
  manifest.json, sw.js, icons/   Configuración PWA
```

---

## 8. Módulos incluidos

- **Dashboard** — indicadores y accesos rápidos.
- **Nueva Cotización** — cliente, tipo de servicio (Transporte / Renta / Aeropuerto), unidad, fechas, chofer, precio manual, cálculo automático de subtotal/IVA/anticipo/saldo y aviso de anticipo.
- **Historial** — abrir, editar, duplicar, descargar PDF y eliminar cotizaciones.
- **Clientes** — base de datos con historial y documentos.
- **Documentos por cliente** — INE, licencia, comprobante de domicilio, contratos, cotizaciones y fotos, guardados como archivos locales (IndexedDB).
- **Contratos** — hoja de inspección autocompletada + firma digital con el dedo o mouse, insertada en el PDF.
- **Catálogo de Unidades** — CRUD de unidades con imagen, tarifas y calendario de disponibilidad (disponible / rentada / reservada / mantenimiento), botón para compartir catálogo.
- **Configuración** — logo, firma, datos de contacto, IVA y reglas de anticipo.

Folios automáticos `COT-0001…` y `CTR-0001…`, PDFs con la identidad de GAR & RAN, y envío por WhatsApp y correo desde el propio dispositivo.
