# Cómo publicar tu quiz en internet

Tu quiz está en la carpeta **`quiz-web`**. Tiene 3 archivos:

- `index.html`: la página.
- `config.js`: los datos de tu Firebase (los llenas en el paso 3).
- `seed.json`: tus 10 preguntas y fotos, para cargarlas con un toque.

Lo haces una sola vez y toma unos 20 a 30 minutos.

---

## Paso 1: Crear tu proyecto en Firebase

1. Entra a **https://console.firebase.google.com** con tu cuenta de Google.
2. Toca **"Crear un proyecto"**. Ponle un nombre, por ejemplo `quiz-lore`.
3. Cuando te pregunte por Google Analytics, **desactívalo**, porque no lo necesitas. Toca **Crear proyecto**.

## Paso 2: Activar la base de datos y el inicio de sesión

**Base de datos:**
1. En el menú de la izquierda: **Compilación → Firestore Database → Crear base de datos**.
2. Elige una ubicación cercana, por ejemplo `southamerica-east1` o `us-east1`.
3. Elige **"Iniciar en modo de producción"** y toca Crear.
4. Ve a la pestaña **Reglas**, borra todo lo que hay y pega esto:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isOwner() {
      return request.auth != null
        && request.auth.token.email == "TU_CORREO@gmail.com"
        && request.auth.token.email_verified;
    }
    match /quiz/{id}   { allow read: if true; allow write: if isOwner(); }
    match /images/{id} { allow read: if true; allow write: if isOwner(); }
    match /results/{uid} {
      allow read: if true;
      allow create: if request.auth != null && request.auth.uid == uid
        && request.resource.data.keys().hasOnly(['name', 'answers', 'at'])
        && request.resource.data.name is string
        && request.resource.data.name.size() > 0
        && request.resource.data.name.size() <= 40;
      allow delete: if isOwner();
    }
    match /wishes/{uid} {
      allow read, delete: if isOwner();
      allow create: if request.auth != null && request.auth.uid == uid
        && request.resource.data.wish is string
        && request.resource.data.wish.size() <= 600;
    }
  }
}
```

5. Cambia **`TU_CORREO@gmail.com`** por el correo de Google con el que vas a entrar a tu panel, y toca **Publicar**.

**Inicio de sesión:**
1. En el menú: **Compilación → Authentication → Comenzar**.
2. En la pestaña **Método de acceso**, activa dos opciones:
   - **Anónimo**: así tus amigos juegan sin crear cuenta.
   - **Google**: así entras tú a tu panel. Elige tu correo como "correo de asistencia" y guarda.

## Paso 3: Conectar la página con Firebase

1. Toca el engranaje ⚙️ junto a "Descripción general del proyecto" y entra a **Configuración del proyecto**.
2. Baja hasta **"Tus apps"** y toca el ícono **`</>`** (Web).
3. Ponle un nombre, por ejemplo `quiz`. **No marques** Firebase Hosting. Toca **Registrar app**.
4. Te aparece un código con `const firebaseConfig = { ... }`. Copia los valores de `apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId` y `appId`.
5. Abre **`quiz-web/config.js`** y reemplaza cada `PEGA_AQUI` por su valor.
6. En ese mismo archivo, cambia **`TU_CORREO@gmail.com`** por el mismo correo que pusiste en las reglas. Guarda.

Estos datos de Firebase no son secretos y pueden estar en la página. Lo que protege tu quiz son las reglas del paso 2.

## Paso 4: Publicar la página con Vercel

**4a. GitHub:** tu quiz ya está en tu repositorio **https://github.com/LorenaFer/lore**, dentro de la carpeta `quiz-web`. Cuando llenes `config.js` (paso 3), sube ese cambio a GitHub. Puedes pedírmelo, o en GitHub entrar a `quiz-web/config.js`, tocar el lápiz ✏️, pegar tus datos y tocar **Commit changes**.

**4b. Conectar con Vercel**
1. Entra a **https://vercel.com** y toca **Sign Up → Continue with GitHub**. Elige el plan **Hobby**, que es gratis.
2. Toca **Add New… → Project**.
3. Busca tu repositorio `lore` y toca **Import**. Si no aparece, toca "Adjust GitHub App Permissions" y dale acceso.
4. En **Framework Preset** elige **Other**.
5. En **Root Directory** toca **Edit** y elige la carpeta **`quiz-web`**. Esto es importante, porque ahí está la página.
6. Toca **Deploy**. En unos segundos te da tu link, por ejemplo `https://lore.vercel.app`.

Si quieres otro nombre en el link: **Settings → Domains → Edit** sobre el dominio `.vercel.app`.

**Último ajuste en Firebase,** para que funcione tu inicio de sesión con Google:
- Ve a **Authentication → Configuración → Dominios autorizados → Agregar dominio** y pega tu dominio de Vercel, por ejemplo `quiz-lore.vercel.app`, sin `https://`.
- Si cambias el nombre del dominio en Vercel, agrega también el nuevo.

## Paso 5: Cargar tus preguntas

1. Abre tu link.
2. Abajo de todo toca **"Soy Lore · entrar a mi panel"** y entra con tu correo.
3. Arriba te aparece **"Cargar mis preguntas y fotos"**. Tócalo y listo: tus 10 preguntas con sus fotos quedan cargadas.
4. Cambia a **"Vista de amigos"** y juega una vez para probar. Después borras tu resultado con la **×** del ranking.

## Paso 6: Compartir 🎉

Manda el link a tus amigos. Ellos solo lo abren y juegan, sin cuenta.

---

### Bueno saber

- **Un intento por persona:** cada dispositivo puede jugar una vez. Si alguien borra los datos de su navegador o juega desde otro celular, podría jugar de nuevo. Si ves algo raro, borra ese resultado con la ×.
- **Tus amigos no ven** las respuestas correctas, los deseos ni tu panel.
- **Si cambias algo de la página** (no las preguntas, sino el diseño o el código), sube el archivo nuevo a GitHub con **Add file → Upload files**. Vercel actualiza la página sola en menos de un minuto. Las preguntas, los resultados y los deseos no se pierden, porque viven en Firebase.
- **Gratis:** el plan gratuito de Firebase y el plan Hobby de Vercel alcanzan de sobra para un quiz entre amigos.
