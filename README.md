# Apuntes Alexa Skills
## Jaime Simeón Palomar Blumenthal - alu0101228587@ull.edu.es

## **Skills**
Los comandos de Alexa tienen un formato con una terminología concreta. Por ejemplo, si quiero ejecutar la skill "Space Facts" digo _Alexa, abre space facts para un solo dato sobre Marte_:

* _Alexa_: Palabra para depertar el dispositivo (_Wake word_).
* _abre_: Acción a ejecutar o lanzamiento (_launch_).
* _space facts_: Skill a ejecutar o nombre de invocación (_invocation name_).
* _para un solo dato sobre_: Opción con la que se lanza la skill (_utterance_).
* _Marte_: Es el slot o parámetro.

Es similar a ejecutar un comando de Shell: _ls -la Documentos_ lanza el comando ls con las opciones -l y -a con el parámetro _Documentos_. No hace falta ni despertar el servicio porque shell siempre está corriendo, ni especificarle que tiene que ejecutar algo porque es el comportamiento predefinido. ls es el servicio a ejecutar, -l, -a las opciones con las que se ejecutará el programa, y Documentos el parámetro.

La filosofía de Alexa consiste en que varias opciones sirvan para la misma acción, o lo que es lo mismo, planear para una intención concreta del usuario y no por opciones. Los slots al igual que las opciones se agrupan según la Intención que nosotros diseñemos.

También existen intenciones predefinidas por Amazon, como AMAZON.CancelIntent para cancelar la ejecución de Skills, o AMAZON.RepeatIntent para repetir frases o Skills.

Las skills se dividen en dos partes:
* **Interacción**: La parte en la que definimos las intenciones, las acciones, las frases concretas que llevarán a opciones, etc...

* **Backend**: El código de las funcionalidades de la skill.

### **Árbol de ficheros**

```sh
/models             #Todo lo relacionado con la interacción
    /id-pa.json     #idioma-país.json (en-US.json, p.ej.) fichero principal de interacción
/lambda/custom      #Todo lo relacionado con el backend
    /index.js       #Fichero principal de backend
```

## **Alexa Developer Console**
Consta de dos partes (interacción y backend) separadas y compilables por separado.

En la parte de interacción definiremos todo lo relacionado con esta de forma gráfica principalmente, pero también podremos editar un fichero JSON si nos sentimos más cómodos donde se sincronizan los cambios.

En el backend toca programar en javascript puro y duro las acciones que desencadena cada uno de nuestras intenciones (_intents_ de aquí en adelante).

```js
const HelloWorldIntentHandler = {
    canHandle(handlerInput) {   //Determina si el input de usuario se maneja con este intent.
        //...
    },
    handle(handlerInput) {
        const speakOutput = 'Hello World!'; //String a decir por Alexa.
        return handlerInput.responseBuiler  //Constructor de respuestas.
            .speak(speakOutput)     //Alexa dice el string.
            .getResponse();         //Espera a una respuesta del usuario.
    },
    //Más handlers
};
```

Finalmente, en el apartado de Test podremos ejecutar nuestra skill y ver qué JSON entra y cuál retorna.

## **Soporte para varios idiomas**
Copiamos el JSON de interacción, vamos a las opciones de idioma, añadimos un idioma nuevo, y reemplazamos el JSON por defecto y por el que teníamos en el portapapeles.

La clave de esto es que los intents por defecto no dependen del idioma en que se compilen. Hay que cambiar son los strings del JSON del backend para que estén en el idioma que se quiera. Además hay que añadir la dependencia "i18next" a _packages.json_ en el backend y darle a **DEPLOY**:

```js
{
    "name": "hello-world",
    "version": "1.1.1",
    //Más datos de la skill
    "dependencies": {
        "ask-sdk-core": "2.6.0",
        //Más dependencias
        "i18next": "15.0.5"
    }
}
```

También se debe separar los strings del resto del código. Esto lo logramos haciendo una tabla hash de objetos e instanciando la dependencia _i18next_ en el fichero _index.js_. Además, tendremos que utilizar un interceptador, que cogeŕa el input del usuario antes de pasarlo a los handlers e identificará el idioma en el que se le está hablando a Alexa:

```js
const Alexa = require('ask-sdk-core');
const i18n = require('i18next');    //Esta es la instanciación de la dependencia.

const languageStrings = {   //Aquí definiremos los mensajes clasificados por idiomas.
    en: {   //Mensajes en inglés
        translation: {
            WELCOME_MSG: "Welcome, you can say Hello or Help."
            HELLO_MSG: "Hello World!"
        }
    },
    es: {   //Mensajes en español
        translation: {
            WELCOME_MSG: "Bienvenido, puedes decir Hola o Ayuda."
            HELLO_MSG: "¡Hola Mundo!"
        }
    }
}

// Manejadores de intents y demás contenido
const launchRequestHandler = {
    //...
    handle(handlerInput) {
        /*Ya no especificamos el string concreto, sino que llamamos a la función t y le pasamos el mensaje adecuado. No le especificamos el idioma porque ya lo sabe.*/
        const speakOutput = handlerInput.t('HELLO_MSG');
        return handlerInput.responseBuiler
            .speak(speakOutput)
            .getResponse();
    },
    //Más handlers.
}

//Definición de las acciones de la función del interceptador.
const localisationRequestInterceptor = {
    process(handlerInput) { //Toma como parámetro el input
        i18n.init({         //Inicializa i18next
            lng: Alexa.getLocale(handlerInput.requestEnvelope),
            resources: languageStrings                          //Utiliza la hash que definimos al principio
        }).then((t) => {    //Define la función t de handlerInput
            handlerInput.t = (...args) => t(...args);
        });
    }
};

//Definición del interceptador
esports.handler = Alexa.SkillBuilders.custom()
    .addRequestHandlers(
        //Lista de todos los handlers que se van a utilizar
    )
    .addRequestInterceptors(
        localisationRequestInterceptor
    )
    //...
```

Un aspecto importante es que, en la tabla hash, debemos utilizar los códigos para los idiomas predefinidos (it, es, en, etc).


## **Slots**

A la hora de crear un intent, tendremos que designar las Utterances. Y si existen variables en ellas, estas son denominadas slots.

```
Mi fecha de nacimiento es el {day} de {month} del {year}.
```

Esta es una utterance para que Alexa recuerde nuestra fecha de nacimiento. Es decir, almacenará las variables _day_, _month_ y _year_.

A estos slots tendremos que asignarles un tipo de dato. También podremos crear uno nuevo, por ejemplo, para _month_.