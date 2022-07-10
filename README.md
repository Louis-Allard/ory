# ory
Test ory

## Assets
 VueJS : https://cli.vuejs.org/guide/creating-a-project.html#vue-create

 Ory :  https://console.ory.sh/projects/f3a9ca3d-5ae0-4cf6-bb3a-752386e9304b/created

## Install
### First step

```
npx @vue/cli create ory-spa --preset "Default (Vue 3)"
cd ory-spa
npm i --save @ory/client
```

### Second step

Avec un projet Vue.js créé, ajoutez la logique Ory à l'application. L'application doit reconnaître si l'utilisateur est connecté et s'il a une session valide.

Selon que l'utilisateur dispose ou non d'une session valide, l'application doit afficher des liens d'inscription et de connexion ou des paramètres de compte et des liens de déconnexion.

Pour implémenter ce comportement, modifiez le composant HelloWorld.vue existant pour inclure la logique appropriée :


```
<template>
  <div class="main">
    <h1>{{ msg }}</h1>

    <div v-if="!session">
      <p>Click on "login" or "Sign Up" below to sign in.</p>
      <li><a :href="basePath + '/ui/login'" data-testid="sign-in">Login</a></li>
      <li>
        <a :href="basePath + '/ui/registration'" data-testid="sign-up"
          >Sign Up</a
        >
      </li>
    </div>

    <h3 v-if="session">Calling <code>toSession()</code></h3>
    <div v-if="session" class="long">
      <p>
        Use the SDK's <code>toSession()</code> call to receive the session
        information, for example the authentication methods used:
      </p>
      <pre><code data-testid='ory-response'>{{ session.authentication_methods }}</code></pre>
    </div>

    <h3 v-if="apiResponse">API Response</h3>
    <div v-if="apiResponse" class="long">
      <p>
        Or make authenticated AJAX calls to your API using <code>fetch()</code>:
      </p>
      <pre><code data-testid='api-response'>{{ apiResponse }}</code></pre>
    </div>

    <h3 v-if="session">Common Actions</h3>
    <ul v-if="session">
      <li><a :href="logoutUrl" data-testid="logout">Logout</a></li>
      <li>
        <a :href="basePath + '/ui/settings'" data-testid="settings">Settings</a>
      </li>
    </ul>

    <h3>Essential Links</h3>
    <ul>
      <li><a href="https://www.ory.sh">Ory Website</a></li>
      <li><a href="https://github.com/ory">Ory GitHub</a></li>
      <li><a href="https://www.ory.sh/docs">Documentation</a></li>
    </ul>
  </div>
</template>

<script>
import { V0alpha2Api, Configuration } from "@ory/client"

// The basePath points to the location of Ory's APIs.
// You can use https://<slug>.projects.oryapis.com/ here because cookies can not
// easily be shared across different domains.
//
// In the next step, we will run a process to mirror Ory's APIs
// on your local machine using the Ory Tunnel at http://localhost:4000
const basePath = process.env.VUE_APP_ORY_URL || "http://localhost:4000"
const ory = new V0alpha2Api(
  new Configuration({
    basePath,
    baseOptions: {
      // Ensures we send cookies in the CORS requests.
      withCredentials: true,
    },
  }),
)

const apiUrl = process.env.VUE_APP_API_URL || "http://localhost:8081"

export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      session: null,
      logoutUrl: null,
      apiResponse: null,
      basePath,
    }
  },
  mounted() {
    // Fetch the session directly from Ory
    ory.toSession().then(({ data }) => {
      this.session = data

      // If the user is logged in, we want to show a logout link!
      ory.createSelfServiceLogoutFlowUrlForBrowsers().then(({ data }) => {
        this.logoutUrl = data.logout_url
      })
    })

    // Or make an authenticated request to your API
    fetch(apiUrl + "/api/hello", {
      // Do not forget to set this - it is required to send the session cookie!
      credentials: "include",
    })
      .then(
        (res) =>
          res.ok &&
          res.json().then((res) => {
            this.apiResponse = res
          }),
      )
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main {
  max-width: 400px;
  margin: 0 auto;
}
</style>
```

### Third step

Run the API

Le modèle de sécurité Ory utilise des cookies HTTP pour gérer les sessions, les jetons et les cookies. Pour ce faire, les API Ory doivent être exposées sur le même domaine que votre application.

Dans le cas de cet exemple, le domaine de votre application, qui s'exécute sur votre machine locale, est localhost.
Info

Lors du développement local, utilisez localhost ou 127.0.0.1, mais pas les deux. Bien que techniquement, cela signifie la même chose, ce sont des noms d'hôte différents.

L'utilisation des deux de manière interchangeable dans votre code peut entraîner des problèmes avec l'origine des cookies HTTP envoyés par l'application. Lorsque cela se produit, les API Ory peuvent ne pas être en mesure de lire les cookies envoyés par votre application.

Pour que votre application s'exécute localement et les API Ory sur le même domaine, utilisez Ory Tunnel - un outil de développement fourni avec Ory CLI. C'est comme un microservice - un serveur d'API d'authentification sur votre domaine !

```
npx @ory/cli tunnel --dev http://localhost:8080

```

Les API Ory sont désormais mises en miroir sur http://localhost:4000. Utilisez cette URL comme baseUrl pour le SDK @ory/client (voir le code HelloWorld.vue ci-dessus). L'indicateur --dev désactive les vérifications de sécurité pour une intégration plus facile et ne doit pas être utilisé lors du déploiement du tunnel dans un environnement intermédiaire.

In case of error with ory tunnel, see https://www.ory.sh/docs/cli/ory-tunnel

### Fourth step

Run the VueJS application

Tout d'abord, configurez votre URL Ory Cloud SDK pour vous connecter aux API Ory dans votre projet Ory Cloud :

Sur Linux:

```
# This is a public Ory Cloud Project.
# Don’t submit any personally identifiable information in requests made with this project.
# Sign up for Ory Cloud at
#
#   https://console.ory.sh/registration
#
# and create a free Ory Cloud Project to see your own configuration embedded in code samples!
export ORY_SDK_URL=https://{your-project-slug-here}.projects.oryapis.com
```

Enfin lancer:

```
npm run serve
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```