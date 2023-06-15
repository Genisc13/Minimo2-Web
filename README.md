#Alcance de Desarrollo:
Bien, después de la reunión que Tuvimos con Toni el pasado 13 de junio, dediqué varias horas a implementar el login y el registro utilizando google, en este readme, como en el README de web, voy a explicar en que lugares apliqué cambios exactamente para poder permitir el login y el registro usando una entidad externa como en google:

Antes de hacer algún cambio, de hicieron unas instalaciones de dependencias:
npm install jwt-decode //Para desencriptar el token de google
npm install uuid //Para la generación de contraseñas y nombres de usuario automática.
npm install @react-oauth/google //Para el manejo que hay detrás de las peticiones a google de las credenciales.

//Y otra cosa más antes de ir al codigo, para que todo esto funcione hay que ir a Google Console, crear una cuenta, crear un proyecto y ahí crear una pantalla de consentimiento, y no solo eso, además hay qe crear las credenciales correspondientes de los clientes de OAuth porque sinó no se permite hacer peticiones de credenciales a google /(Además que hay que hacer varias cosas como por ejemplo añadir las urls permitidas)

- /lplan-web/src/pages/home.page
  Dentro de home.page.tsx se han hecho casi todos los cambios necesarios para conseguir que se pueda hacer el login y el Registro en google:

  Primero añadimos los imports:
  import { GoogleLogin } from "@react-oauth/google";
  import { v4 as uuidv4 } from "uuid";
  import jwt_decode from "jwt-decode";
  //Primero creamos la función para generar la contraseña:

  const [password, setPassword] = useState("");
  
  const generatePassword = () => {
  const newPassword = uuidv4();
  setPassword(newPassword);
  };

    Luego creamos la función: handleGoogleLoginSuccess

    Esta función que es bastante larga se encarga de hacer el login y el Registro del usuario, primero de todo busca al usuario con el email (Función añadida al Backend) proporcionado por el Google Login (Que lo veremos en el return como una etiqueta que es básicamente un botón con toda la lógica de las peticiones al servidor de google), si lo encuentra, se hace el login en nuestra aplicación usando el loginGoogle de nuestro service que se encarga de hacer la petición a Backend.
    Por otro lado, si no se encuentra el usuario, se hace primero un registro con unos datos por defecto y una contraseña y usuario generados aleatoriamente, se hace el Registro y una vez hecho el Registro se hace inmediatamente un Login con la logica de antes, lo que lleva a que el usuario entre en la aplicación en ambos casos.
    Esto nos da un Login/Registro con google con un solo botón

    Además de añadir esta función, en el return de la page se añade:
    GoogleLogin onSuccess={handleGoogleLoginSuccess} onError={() => {console.log("Login Failed");}}
    
    Dentro de un div
    Lo que nos genera el botón encargado de hacer la autenticación de Google.

- /lplan-web/src/index.tsx
    En el root.render, se ha añadido el GoogleOAuthProvider con el id de cliente de web que previamente se ha creado en el Google Console.

- /lplan-web/src/services/user.service.ts
    Se creó la siguiente función para hacer lo explicado anteriormente:
    static async getPersonByEmail(email: string) {
        try {
        const response = await axios.get(API_URL +"/google/check/" + email, { headers: authHeader() });
        return response;
        } catch (error) {
        console.error("Error when obtaining person:", error);
        throw error;
        }
    }
- /lplan-web/src/services/auth.service.ts
    Se creó la siguiente función para hacer lo explicado anteriormente

    static async loginGoogle(auth: Auth) {
        try {
        const response = await axios.post(API_URL + "/loginfrontendgoogle", auth);
        return response;
        } catch (error) {
        console.error('Error during login:', error);
        throw error;
        }
    }

