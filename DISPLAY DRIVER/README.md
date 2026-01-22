# Display driver Ver1.00
　Este es un controlador para gestionar las fuentes de caracteres mostradas en el entorno FX-DOS.<br>
Aunque es posible mostrar texto sin él, instalar este controlador permite mejorar la velocidad de visualización de caracteres, lograr pantallas de alta densidad y habilitar entornos en japonés, entre otras funciones.<br><br>
　Este controlador integra las funcionalidades de los dos controladores anteriores, "SMALLFNT" y "KNJFNT8", y además añade extensiones de funciones. Por favor, úselo reemplazando a esos controladores anteriores.<br><br>
　Las mejoras respecto a los controladores anteriores son: el aumento a 10 modos de pantalla, la capacidad de cambiar el modo de pantalla sin reiniciar DOS, y la posibilidad de combinar gráficos y visualización de texto usando G-API.<br>
En cuanto a la visualización de caracteres kanji, anteriormente no había espaciado entre caracteres, lo que dificultaba un poco la lectura. Por ello, se ha preparado un modo de pantalla con espaciado (modo de pantalla "9"). (Se añade un espacio de 2 píxeles entre caracteres de ancho completo).<br><br>

※ Este software utiliza internamente las funciones del gestor de fuentes y de G-API, por lo que asegúrese de tenerlos preparados previamente.<br><br>

※ Incluye listado de código fuente.

