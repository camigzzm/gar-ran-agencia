# GAR & RAN Agencias — Plataforma Web (PWA)

Aplicación web progresiva para la gestión de cotizaciones, clientes, contratos, catálogo de unidades y transporte ejecutivo de **GAR & RAN Agencias**.

Construida con **Next.js 14 (App Router) + TypeScript + Tailwind CSS + Firebase** (Auth, Firestore, Storage). Lista para producción en **Vercel**, instalable como app en Android e iPhone sin App Store.

---

## 1. Requisitos previos

- Node.js 18 o superior
- Una cuenta de [Firebase](https://console.firebase.google.com/)
- Una cuenta de [Google Cloud](https://console.cloud.google.com/) para la API de Google Maps (puede ser el mismo proyecto de Firebase)
- Una cuenta de [GitHub](https://github.com/) y de [Vercel](https://vercel.com/)

---

## 2. Configurar Firebase

1. Crea un proyecto en [Firebase Console](https://console.firebase.google.com/).
2. **Authentication** → pestaña "Sign-in method" → habilita **Correo electrónico/Contraseña**.
3. **Firestore Database** → crea la base de datos (modo producción).
4. **Storage** → actívalo (modo producción).
5. En **Configuración del proyecto → General → Tus apps**, agrega una app web y copia los valores del `firebaseConfig` (los necesitarás para el archivo `.env.local`).

### Reglas de seguridad

Este proyecto incluye `firestore.rules` y `storage.rules` listas para usar. Publícalas desde la [Firebase Console](https://console.firebase.google.com/) (Firestore → Reglas / Storage → Reglas) copiando el contenido de cada archivo, o con la CLI de Firebase:

```bash
npm install -g firebase-tools
firebase login
firebase init   # selecciona Firestore y Storage, usa los archivos ya incluidos
firebase deploy --only firestore:rules,storage:rules
```

### Crear el primer usuario administrador

Los usuarios nuevos solo pueden crearse desde la pantalla **Configuración → Usuarios** dentro de la app, pero esa pantalla requiere que ya exista un administrador. Para el primer usuario:

1. Firebase Console → **Authentication** → "Add user" → captura correo y contraseña.
2. Copia el **UID** generado.
3. Firebase Console → **Firestore Database** → crea manualmente la colección `users` con un documento cuyo ID sea ese mismo UID, con estos campos:

   | Campo | Tipo | Valor |
   |---|---|---|
   | `email` | string | el correo que usaste |
   | `name` | string | nombre del administrador |
   | `role` | string | `admin` |
   | `active` | boolean | `true` |
   | `createdAt` | number | `1751500800000` (o cualquier timestamp) |

4. Inicia sesión con ese usuario en la app — desde ahí ya podrás crear al resto del equipo (con rol `admin` o `agente`) desde **Configuración**.

---

## 3. Configurar Google Maps (módulo "Calculadora Google Maps")

1. En [Google Cloud Console](https://console.cloud.google.com/google/maps-apis), habilita:
   - **Maps JavaScript API**
   - **Places API**
   - **Distance Matrix API**
2. Crea una **API key** y restríngela por dominio (tu dominio de Vercel y `localhost` para desarrollo).

---

## 4. Variables de entorno

Copia `.env.example` a `.env.local` y completa los valores:

```bash
cp .env.example .env.local
```

```
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=
```

---

## 5. Ejecutar en local (Visual Studio Code)

```bash
npm install
npm run dev
```

Abre [http://localhost:3000](http://localhost:3000). Inicia sesión con el usuario administrador que creaste en el paso 2.

> Si `npm install` falla por conflictos de dependencias (`ERESOLVE`), ejecuta `npm install --legacy-peer-deps`. El `package.json` ya incluye un `overrides` para evitar este problema en la mayoría de los casos.

---

## 6. Subir el proyecto a GitHub

```bash
git init
git add .
git commit -m "GAR & RAN Agencias - versión inicial"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/garran-app.git
git push -u origin main
```

> El archivo `.gitignore` ya excluye `node_modules`, `.env*.local` y los archivos de build, así que tus credenciales nunca se suben al repositorio.

---

## 7. Desplegar en Vercel

1. Entra a [vercel.com/new](https://vercel.com/new) e importa el repositorio de GitHub.
2. Framework Preset: **Next.js** (se detecta automáticamente).
3. En **Environment Variables**, agrega las mismas variables de tu `.env.local`.
4. Haz clic en **Deploy**.
5. Cuando termine, obtendrás una URL pública (por ejemplo `https://garran-app.vercel.app`) — actualiza la restricción de dominio de tu API key de Google Maps con esta URL.

---

## 8. Instalar como app (PWA) sin App Store

**Android (Chrome):** abre la URL de Vercel → menú (⋮) → "Instalar aplicación" / "Agregar a pantalla de inicio".

**iPhone (Safari):** abre la URL de Vercel → botón compartir (□↑) → "Agregar a pantalla de inicio".

La app se instalará con ícono propio y funcionará en modo standalone (pantalla completa, sin barra de navegador).

---

## 9. Estructura del proyecto

```
src/
  app/                     Rutas (App Router de Next.js)
    login/
    (protected)/           Grupo de rutas protegidas por sesión
      dashboard/
      cotizaciones/
      clientes/
      contratos/
      unidades/
      mapa/
      configuracion/
  components/
    ui/                    Sistema de diseño reutilizable
    layout/                Sidebar, BottomNav, Header, AppShell
    quotes/ clients/ units/ contracts/ maps/ settings/ signature/
  lib/
    firebase.ts            Inicialización de Firebase
    auth-context.tsx        Contexto de autenticación y roles
    types.ts                Tipos de datos compartidos
    quote-calc.ts           Lógica de cálculo de cotizaciones
    services/                Capa de acceso a Firestore/Storage
    pdf/                     Generación de PDFs (jsPDF)
public/
  manifest.json, sw.js, icons/   Configuración PWA
firestore.rules, storage.rules   Reglas de seguridad de Firebase
```

---

## 10. Módulos incluidos

- **Dashboard** — indicadores y accesos rápidos.
- **Nueva Cotización** — cliente, tipo de servicio (Transporte / Renta / Aeropuerto), unidad, fechas, chofer, precio manual, cálculo automático de subtotal/IVA/anticipo/saldo y aviso de anticipo.
- **Historial** — abrir, editar, duplicar, descargar PDF y eliminar cotizaciones.
- **Clientes** — base de datos con historial y documentos.
- **Documentos por cliente** — INE, licencia, comprobante de domicilio, contratos, cotizaciones y fotos en Firebase Storage.
- **Contratos** — hoja de inspección autocompletada + firma digital con el dedo o mouse, insertada en el PDF.
- **Catálogo de Unidades** — CRUD de unidades con imagen, tarifas y calendario de disponibilidad (disponible / rentada / reservada / mantenimiento), botón para compartir catálogo.
- **Calculadora Google Maps** — distancia, tiempo, gasolina estimada y precio sugerido editable.
- **Configuración** — logo, firma, datos de contacto, IVA, reglas de anticipo y gestión de usuarios (solo administradores).

Folios automáticos `COT-0001…` y `CTR-0001…`, PDFs con la identidad de GAR & RAN, envío por WhatsApp y correo, y respaldo en la nube vía Firebase.
