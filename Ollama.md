# Proyecto 3. Ollama + RAG + Asistente sobre implementación de medidas de seguridad

## 1. Objetivo
He montado un asistente que ayuda con problemas de phishing. El usuario escribe lo que le ha pasado, pulsa un botón y recibe los pasos para solucionarlo. Todo funciona en local con Ollama desde PowerShell de Windows, y se usa desde una página web.

## 2. Instalación de Ollama
Instalé Ollama en Windows y lo uso desde PowerShell.

<img width="411" height="49" alt="image" src="https://github.com/user-attachments/assets/f88fe974-cc01-4d80-a290-e6e3067b32e7" />

## 3. Modelo
Usé qwen3.5:4b como modelo base. Para que contestara más rápido creé un modelo propio, "rapido", con un Modelfile donde reduje num_ctx, num_predict y temperature. Lo abro con el modo de pensar desactivado.

<img width="1094" height="283" alt="image" src="https://github.com/user-attachments/assets/f1f2135c-fa85-4225-b07c-829e5dfff005" />
<img width="524" height="60" alt="image" src="https://github.com/user-attachments/assets/81f4f01b-a7f5-4930-8e45-918bca1a92db" />

El archivo está en [Modelfile](Modelfile).

## 4. Modelo de embeddings
Para el RAG descargué `nomic-embed-text`, que convierte los textos en vectores y permite buscar los más parecidos a la pregunta.

<img width="637" height="133" alt="image" src="https://github.com/user-attachments/assets/bc1a9ad5-d4ce-4667-a978-8ab9d3b028ed" />

## 5. Base de conocimiento (RAG)
Escribí 8 fragmentos sobre phishing. Cuando el usuario pregunta, la página busca los 2 más parecidos y se los pasa al modelo como contexto.

```javascript
const KB = [
 "Si hiciste clic en un enlace sospechoso pero no introdujiste datos: cierra la pestaña, no descargues nada, ejecuta un análisis antivirus y avisa a IT o seguridad.",
 "Si introdujiste tu contraseña en una web falsa: cambia la contraseña inmediatamente desde la web oficial, activa la verificación en dos pasos, revisa los inicios de sesión recientes y cierra todas las sesiones abiertas. Si usas esa contraseña en otros sitios, cámbiala también allí.",
 "Si abriste un archivo adjunto sospechoso: desconecta el equipo de la red (Wi-Fi y cable), no lo apagues, avisa a IT y no abras más archivos hasta que lo revisen. Ejecuta un análisis completo con el antivirus.",
 "Si diste datos bancarios o de tarjeta: llama a tu banco para bloquear la tarjeta o cuenta, revisa los movimientos recientes, reclama los cargos no reconocidos y presenta denuncia si hubo pérdida de dinero.",
 "Cómo reportar un phishing: no respondas al correo, no reenvíes el mensaje a compañeros. Usa el botón de reportar phishing del cliente de correo o reenvíalo como adjunto al equipo de seguridad. Después elimínalo.",
 "Cómo reconocer un correo de phishing: remitente que no coincide con la empresa real, urgencia o amenazas, errores de redacción, enlaces cuyo destino real (al pasar el ratón) no es el oficial, y peticiones de contraseñas o datos personales.",
 "Medidas de prevención: usar verificación en dos pasos, un gestor de contraseñas, no abrir adjuntos inesperados, comprobar la URL antes de iniciar sesión, mantener el sistema actualizado y formar a los empleados con simulacros de phishing.",
 "Si recibes un SMS (smishing) o una llamada (vishing) sospechosa: no pulses enlaces, no des códigos recibidos por SMS, cuelga y llama tú al número oficial de la entidad."
];
```

## 6. Prompt del asistente
El prompt le dice al modelo que conteste solo con el contexto que recibe, que no invente pasos y que use siempre el mismo formato: gravedad, pasos y prevención.

```
Eres un asistente de ciberseguridad especializado en incidentes de phishing. Hablas en español, con un tono claro y tranquilizador, para usuarios sin conocimientos técnicos.

REGLAS:
1. Responde SOLO con la información del CONTEXTO. Si el contexto no cubre la situación, dilo y recomienda contactar con el equipo de seguridad o de IT.
2. No inventes pasos, enlaces ni herramientas que no aparezcan en el contexto.
3. Si el usuario indica que ya hizo clic, introdujo credenciales o abrió un adjunto, pon primero las acciones urgentes.
4. Nunca pidas contraseñas ni datos personales al usuario.

FORMATO DE RESPUESTA (breve):
- Gravedad: Baja / Media / Alta (una frase de por qué)
- Pasos a seguir: lista numerada, máximo 6 pasos, cada uno en una línea
- Cómo evitarlo en el futuro: 1 o 2 consejos
```

## 7. Página web
La página es un único archivo, [index.html](index.html), hecho con HTML y JavaScript. Llama a la API local de Ollama (`localhost:11434`) para hacer la búsqueda del RAG y generar la respuesta. La sirvo con `python -m http.server 8000` y la abro en `http://localhost:8000`.

<img width="879" height="389" alt="image" src="https://github.com/user-attachments/assets/b21cab3e-6ea4-4651-a0ad-9d15ecebe841" />
<img width="1789" height="1028" alt="image" src="https://github.com/user-attachments/assets/173e2e95-03a7-477b-9311-8cef92635268" />

## 8. Pruebas
**Prueba 1:** "Hice clic en un enlace de un correo del banco y puse mi contraseña"

<img width="841" height="886" alt="image" src="https://github.com/user-attachments/assets/ea53fe0e-59b1-4d77-abbe-622b41598553" />

La gravedad y los pasos son correctos (bloquear la tarjeta, revisar movimientos, denunciar), pero no dice que hay que cambiar la contraseña. El RAG cogió el fragmento de datos bancarios porque la pregunta nombra al banco.

**Prueba 2:** "Abrí un archivo adjunto de un correo raro y ahora el ordenador va lento"

<img width="811" height="874" alt="image" src="https://github.com/user-attachments/assets/b058c142-97ff-4b7d-8511-8734d356418e" />

El RAG recuperó el fragmento de adjuntos sospechosos. La respuesta dice que hay que desconectar el equipo de la red, no apagarlo, pasar el antivirus y avisar a IT.

**Prueba 3:** "Puse mi contraseña de Instagram en una web falsa"

<img width="801" height="928" alt="image" src="https://github.com/user-attachments/assets/38dfc5e3-d43c-4cbd-a59e-101780ec0b19" />

Aquí el RAG recuperó el fragmento de contraseñas: cambiarla, activar la verificación en dos pasos, revisar los inicios de sesión, cerrar sesiones y cambiarla en otros sitios.

**Error durante las pruebas:** al principio la página mostraba "(intermediate value) is not iterable". Pasaba porque no había descargado `nomic-embed-text` y Ollama no devolvía los vectores. Lo solucioné con `ollama pull nomic-embed-text`.

## 9. Conclusiones
El asistente funciona en local. El RAG busca los fragmentos más parecidos al problema y el modelo "rapido" redacta la respuesta. Como las respuestas cambian según la pregunta, se ve que la búsqueda funciona.

Limitaciones:
- La base de conocimiento es pequeña y puede recuperar un fragmento poco adecuado si la pregunta mezcla temas, como en la Prueba 1.
- El modelo es pequeño y tiene la respuesta limitada en longitud para que vaya rápido.

Mejoras posibles:
- Añadir más fragmentos a la base de conocimiento.
- Recuperar 3 fragmentos en vez de 2.
- Convertir la página en una extensión de Chrome.
