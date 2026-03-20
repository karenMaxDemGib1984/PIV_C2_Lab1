Planteamiento del problema:

En la actualidad, la mayoría de los usuarios utilizan
contraseñas débiles, predecibles o repetidas en múltiples 
plataformas (como "123456" o nombres de mascotas). Esto 
los hace vulnerables a ataques de ciberdelincuencia. La 
falta de una herramienta sencilla que genere y gestione 
criterios de aleatoriedad impide que las personas mantengan 
mínimos lienamientos de seguridad digital.

 ¿Qué es Vue.js y cuál es su función en esta página?
Vue.js es un framework progresivo de JavaScript orientado a construir interfaces de usuario. 
Su característica principal es la reactividad: cuando los datos cambian, la interfaz se actualiza
automáticamente sin necesidad de manipular el DOM manualmente.
En esta aplicación, Vue.js cumple la función de mantener sincronizados los controles del formulario
(slider, checkboxes, botones) con la contraseña generada y sus métricas. Cada vez que el usuario ajusta una opción, 
Vue detecta el cambio y regenera la contraseña y los indicadores de fuerza en tiempo real.


Variables reactivas utilizadas
El código usa ref y reactive para declarar estado reactivo:

La variable options es de tipo reactive({...})  su Objeto que agrupa todas las opciones de configuración: longitud, tipos de caracteres, sin ambiguos, sin repetidos.
Su funcion  La variable passwor es de tipo dref('')  Almacena la contraseña generada actualmente.

La variable copied es de tipo ref(false) su funcion es Controla si se muestra el ícono/mensaje de "copiado" tras copiar al portapapeles.

La diferencia entre las siguientes directivas utilizadas en nuestro proyecto: v-bind y v-model

La diferencia entre ambos es que ref se usa para valores simples (string, boolean), mientras que reactive se usa para objetos con múltiples propiedades relacionadas.

Diferencia entre v-bind y v-model
v-bind es una vinculación unidireccional: lleva datos desde JavaScript hacia el HTML. 
Solo lee el estado y lo aplica al atributo del elemento. Ejemplo en el código: <div class="bar-fill" :style="{ width: strength.percent + '%', background: strength.color }" />


Aquí :style (abreviatura de v-bind:style) aplica el color y ancho de la barra según el valor calculado, pero el usuario no puede modificar ese dato desde la barra.
v-model es una vinculación bidireccional: sincroniza el valor del elemento con la variable en ambas direcciones. Si el usuario cambia el control, 
la variable se actualiza; si la variable cambia por código, el control refleja el nuevo valor. Ejemplo: <input type="range" v-model.number="options.length" @input="generate" />

Al mover el slider, options.length se actualiza automáticamente y viceversa.


Ejemplo de evento utilizado:

El evento más claro es @click en el cuadro de contraseña:
<div class="pw-box" @click="copyPassword">
Cuando el usuario hace clic sobre el área donde se muestra la contraseña, se ejecuta la función copyPassword(), 
que escribe el texto en el portapapeles del sistema usando la API navigator.clipboard. También se usa @input en el slider y @change en los checkboxes para llamar a generate() 
cada vez que cambia una opción.



Uso de v-for en la aplicación
v-for se utiliza para renderizar la lista de checkboxes de forma dinámica, sin repetir código HTML manualmente para cada opción:
<label v-for="c in checkboxes" :key="c.key" class="check-item">  <input type="checkbox" v-model="options[c.key]" @change="generate" />{{ c.label }} </label>

En lugar de escribir seis bloques <label> idénticos, se define el array checkboxes con los objetos { key, label } y Vue genera automáticamente un elemento por cada entrada.
Esto hace el código más limpio y fácil de mantener: si se quiere agregar una opción nueva, solo se añade un objeto al array.

Uso de v-if en la aplicación

Aunque el código no usa v-if explícitamente con esa directiva, la lógica condicional equivalente aparece en las funciones reactivas y en expresiones ternarias dentro del template: 
{{ copied ? '✅ Copiado!' : 'Copiar contraseña' }}

Esto resuelve el problema de retroalimentación visual al usuario: cuando se copia la contraseña, el botón cambia su texto temporalmente para confirmar la acción, y luego de 1.8
segundos regresa al texto original. Sin esta condición, el usuario no sabría si la operación fue exitosa.

Validación de datos en la aplicación

La validación ocurre en la función generate(), en esta línea:
javascriptif (!chars) chars = 'abcdefghijklmnopqrstuvwxyz'

Este es el caso donde el usuario desmarca todos los checkboxes, lo que dejaría el string chars vacío y causaría un error al intentar generar la contraseña. 
La validación detecta ese estado inválido y asigna un conjunto de caracteres por defecto para que la aplicación nunca falle.

También existe validación implícita en el slider con los atributos min="8" y max="64", que impiden que el usuario ingrese una longitud fuera del rango permitido.

Validar es importante porque protege tanto la experiencia del usuario como la integridad del sistema: sin validación, un estado vacío provocaría que generate() 
entrara en un bucle sin caracteres válidos, generando una contraseña vacía o lanzando un error en consola que rompería la interfaz.
<img width="925" height="2152" alt="image" src="https://github.com/user-attachments/assets/a494108d-94ad-4aaa-9c42-3b8ec3305ed5" />

Yolanda Isabel Marroquin Ulloa SMSS047424
Karen Esmeralda Portillo Portillo SMSS202223


