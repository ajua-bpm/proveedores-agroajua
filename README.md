# Portal de Proveedores AGROAJUA

Portal aislado para productores mexicanos. Cada productor entra con email + clave y ve solamente sus cotizaciones.

**URL producción:** https://proveedores.agroajua.com

## Arquitectura
- HTML/JS standalone, sin frameworks
- Firebase Auth (Email/Password) — separado del sistema interno
- Firebase Firestore proyecto `ajuabmp`, colección `cotizacionesImport`
- Filtro: `where('productorEmail', '==', currentUser.email)`
- Deploy: Vercel (separado de `app.agroajua.com`)

## Setup inicial (una vez)

### 1. Firebase Console
- Authentication → Sign-in method → Email/Password → Habilitar
- Authentication → Users → Add user para cada productor

### 2. Firestore Security Rules
Agregar en `firestore.rules` del proyecto `ajuabmp`:
```
match /cotizacionesImport/{docId} {
  allow read: if request.auth != null &&
              (request.auth.token.email == resource.data.productorEmail ||
               request.auth.token.email == 'agroajua@gmail.com');
  allow update: if request.auth != null &&
                request.auth.token.email == resource.data.productorEmail;
  allow create, delete: if request.auth.token.email == 'agroajua@gmail.com';
}
```

### 3. DNS (GoDaddy)
```
CNAME  proveedores  →  cname.vercel-dns.com
```

### 4. Vercel
```
npx vercel deploy --prod --yes
```
Luego en Vercel dashboard: Settings → Domains → agregar `proveedores.agroajua.com`

## Crear cuenta de productor

Opcion A — Manual desde Firebase Console:
1. Authentication → Add user
2. Email del productor + clave temporal
3. Productor entra y usa "Olvidé mi clave" si quiere cambiarla

Opcion B — Desde app.agroajua.com (módulo Admin → Cuentas Proveedores, pendiente):
- Botón "Crear cuenta productor" genera email + clave temporal y envía link de reset

## Flujo del productor

1. Productor entra a proveedores.agroajua.com con su email + clave
2. Ve solo sus cotizaciones (filtro Firestore por email)
3. Abre una cotización, ve productos con máximo MXN/unidad calculado
4. Ingresa su propuesta MXN/u por producto
5. Botones: Guardar propuesta | Aceptar | Rechazar
6. AGROAJUA admin (Ricardo) ve TODAS las cotizaciones desde app.agroajua.com

## Aislamiento

- Productor NO puede ver: comisión AGROAJUA, márgenes, gastos detallados, otros productores
- Productor NO tiene acceso a app.agroajua.com (auth separada, Firestore rules estrictas)
- Admin (`agroajua@gmail.com`) ve todo desde el sistema interno
