# Manual del Programador de FX-DOS

## Capítulo de Solicitudes de Función

　Este documento explica la información necesaria al crear programas para FX-DOS, especialmente lo relacionado 
con las solicitudes de función (function requests).<br>
　Se asume que el lector posee un conocimiento suficiente de MS-DOS. Se recomienda a quienes no lo tengan que 
preparen material de referencia sobre MS-DOS por separado.<br><br>

## １．Llamadas al Sistema

　Cuando FX-DOS inicia en frío (cold start), varias interrupciones de software (software interrupts) quedan 
disponibles. Estas interrupciones de software se denominan llamadas al sistema (system calls), y generalmente 
son utilizadas por las aplicaciones para acceder a las funcionalidades de FX-DOS.<br>
<pre>
        INT 21H ･･･ Solicitud de Función (Function Request)
        INT 23H ･･･ Dirección de Escape con [BRK]
        INT 25H ･･･ Lectura Absoluta de Disco (Absolute Disk Read)
        INT 26H ･･･ Escritura Absoluta de Disco (Absolute Disk Write)
        INT 29H ･･･ Salida Directa a Consola (Direct Console Output)
        INT 2BH ･･･ Entrada del FEP
        INT 2CH ･･･ Acceso Directo a Archivos (Direct File Access)
        INT 30H ･･･ Información de Estado del FEP
</pre><br>

  ◎Solicitud de Función (INT 21H)<br>
<pre>
      Entrada: AH = Número de función
               Otros según la función llamada.
      Salida:  Según la función llamada.
      Registros Afectados: AX
      Explicación: Proporciona la mayoría de las funcionalidades ofrecidas por FX-DOS, como operaciones de archivos y
                   entrada/salida de datos.
</pre><br>

  ◎Dirección de Escape con [BRK] (INT 23H)<br>
<pre>
      Entrada: Ninguna
      Salida:  Ninguna
      Explicación: Especifica la dirección de la rutina ejecutada al presionar [BRK]. Por defecto, cierra todos los
                   archivos y vuelve a la línea de comandos, pero el usuario puede proporcionar su propia rutina.
</pre><br>

  ◎Lectura/Escritura Absoluta de Disco (INT 25H/26H)<br>
<pre>
      Entrada: AL = Número de unidad (E0=10, E1=11, ..., E5=15)
               DS:BX = Dirección de transferencia de datos
               CX = Número de sectores
               DX = Número de sector inicial
      Salida:  Cy, AL = Código de error
      Registros Afectados: AX, BX, CX, DX, BP, DI, SI
      Explicación: Intercambia datos con unidades extendidas por sectores. Esta función solo puede usarse con unidades
                   extendidas y solo si el controlador de dispositivo la soporta.
</pre><br>

  ◎Salida Directa a Consola (INT 29H)<br>
<pre>
      Entrada: AL = Carácter a mostrar
      Salida:  Ninguna
      Registros Afectados: Ninguno
      Explicación: Envía datos directamente a la rutina de visualización de caracteres. Se conecta directamente al
                   controlador de visualización de texto, permitiendo una salida muy rápida.
</pre><br>

  ◎Entrada del FEP (INT 2BH)<br>
<pre>
      Entrada: Ninguna
      Salida:  AL = Carácter ingresado
               AH = Número de caracteres acumulados en el buffer
      Registros Afectados: AX
      Explicación: Lee datos desde el controlador de entrada de teclado (se puede usar incluso sin FEP). Espera si no
                   hay datos de entrada.
</pre><br>

  ◎Acceso Directo a Archivos (INT 2CH)<br>
<pre>
      Entrada: DS:DX = Nombre de archivo en formato FCB
      Salida:  DX:AX = Tamaño del archivo
               DS:SI = Dirección inicial del archivo
               Cy, AX = Código de error (2, 5, 15)
      Registros Afectados: AX, BX, CX, DX, BP, DI, SI, DS, ES
      Explicación: Obtiene la dirección de inicio de la parte de datos de un archivo en la unidad básica. Se usa para
                   acceder a archivos a alta velocidad, pero su uso indebido puede dañar otros archivos.
</pre><br>

  ◎Información de Estado del FEP (INT 30H)<br>
<pre>
      Entrada: AH = 0 Obtener versión (AX)
                   1 Obtener modo de entrada (AL)
                   2 Establecer modo de entrada (AL=0: ANK, FF: Kanji)
      Salida:  AX = Versión (si entrada AH=0)
               AL = Modo de entrada (0: ANK, FF: Kanji) (si entrada AH=1)
      Registros Afectados: AX
      Explicación: Consulta o configura el estado del FEP (Front End Processor). No se puede usar si no hay FEP, pero
                   esto permite verificar si un FEP está registrado.
</pre><br>


## ２．Solicitudes de Función (Function Requests)

　La mayor parte de los servicios proporcionados por FX-DOS están disponibles a través de estas solicitudes de 
función.<br>
　Están basadas fundamentalmente en las funciones de MS-DOS, pero incluyen algunas con comportamiento ligeramente 
diferente y otras exclusivas de FX-DOS.<br>
　Tenga en cuenta que los códigos de error devueltos ante problemas pueden diferir de los de MS-DOS.<br><br>

  ※ Nota: Básicamente, AX se modifica en estas funciones.<br>
  ※ Las funciones marcadas con «*» tienen diferencias de comportamiento respecto a MS-DOS.<br><br>

<pre>
      00H ･･･ Terminar Proceso                    3AH ････ Eliminar Subdirectorio
      01H ･･･ Entrada de Tecla (con eco)          3BH ････ Cambiar Directorio Actual
      02H ･･･ Salida a Pantalla                   3CH ････ Crear Archivo
      03H ･･･ Salida a AUX                        *3DH ････ Abrir Archivo
      04H ･･･ Entrada desde AUX                   3EH ････ Cerrar Archivo
      05H ･･･ Salida a PRN                        3FH ････ Leer Archivo
      06H ･･･ E/S Directa de Consola              40H ････ Escribir Archivo
      07H ･･･ Entrada Directa de Consola          41H ････ Eliminar Archivo
      08H ･･･ Entrada de Tecla                    42H ････ Mover Puntero de Archivo
      09H ･･･ Salida de Cadena                    *43H ････ Obtener/Establecer Atributos
      0AH ･･･ Entrada de Cadena                   *4400H ･･ Obtener Info. de Dispositivo
      0BH ･･･ Verificar Teclado                   4406H ･･ Obtener Estado de Entrada
      0CH ･･･ Vaciar Buffer y Entrar Tecla        4407H ･･ Obtener Estado de Salida
      0EH ･･･ Establecer Unidad Actual            *440EH ･･ Obtener Mapa de Unidades
      19H ･･･ Obtener Unidad Actual               47H ････ Obtener Directorio Actual
      1AH ･･･ Establecer Dirección DTA            48H ････ Asignar Bloque de Memoria
     *25H ･･･ Establecer Vector Int.              4AH ････ Cambiar Tamaño Bloque Mem.
      2AH ･･･ Obtener Fecha                       4BH ････ Ejecutar Programa
      2BH ･･･ Establecer Fecha                    *4CH ････ Terminar Proceso
      2CH ･･･ Obtener Hora                        4DH ････ Obtener Código de Salida
      2DH ･･･ Establecer Hora                     4FH ････ Buscar Archivo
      2FH ･･･ Obtener Dirección DTA               4FH ････ Continuar Búsqueda Archivo
     *30H ･･･ Obtener Número de Versión           52H ････ Obtener Dirección Var. Entorno
     *31H ･･･ Terminar y Residentar               *56H ････ Renombrar Archivo
     *35H ･･･ Obtener Vector de Interrup.         57H ････ Obtener/Establecer Fecha/Hora
      36H ･･･ Obtener Espacio Libre en Disco      5BH ････ Crear Archivo Nuevo
      39H ･･･ Crear Subdirectorio
     *80H ･･･ Formatear Disco                     *88H ･･･ Obtener Día de la Semana
     *81H ･･･ Obtener Cadena de Mensaje           *89H ･･･ Registrar Variable de Entorno
     *82H ･･･ Establecer Volumen Click Tecla      *8AH ･･･ Obtener Variable de Entorno
     *83H ･･･ Obtener Volumen Click Tecla         *8BH ･･･ Asignar FFE
     *84H ･･･ Obtener Tamaño Área Máquina         *8CH ･･･ Especificar Búfer Histórico CON
     *85H ･･･ Nombre Archivo a Formato Est.       *8DH ･･･ Abrir AUX
     *86H ･･･ Nombre Archivo a Formato FCB        *8EH ･･･ Cerrar AUX
     *87H ･･･ Ejecutar Archivo Batch              *8FH ･･･ Analizar Ruta de Directorio
</pre><br>

  ◎Establecer Vector de Interrupción (AH=25H)<br>
<pre>
      Entrada: AL = Número de interrupción
               DS:DX = Vector
      Salida:  Ninguna
      Explicación: Si la dirección del vector es mayor que la dirección inicial del área de archivos «F0» y menor que
                   el final de la RAM, se procesa mediante relocalización automática (self-relocation).
</pre><br>

  ◎Obtener Número de Versión (AH=30H)<br>
<pre>
      Entrada: Ninguna
      Salida:  AX = 0A03H
               BX = 0FF00H
               CL = Parte entera de la versión
               CH = Parte decimal de la versión
      Explicación: Para la versión 1.23, devuelve «CL=1, CH=23». En MS-DOS, CX devolvería «0» como número de usuario.
                   Verificando esto se puede distinguir si el SO es FX-DOS o MS-DOS.
</pre><br>

  ◎Terminar y Dejar Residente (AH=31H)<br>
<pre>
      Entrada: AL = Código de salida
      Salida:  Ninguna
      Explicación: Existe por compatibilidad con MS-DOS. Internamente funciona exactamente igual que «Terminar Proceso
                   (AH=4CH)».
</pre><br>

  ◎Obtener Vector de Interrupción (AH=35H)<br>
<pre>
      Entrada: AL = Número de interrupción
      Salida:  AL = Flag de enlace (0FFH: Enlace DOS)
               ES:BX = Vector
      Explicación: «Enlace DOS» indica que la rutina de manejo de la interrupción reside dentro del sistema DOS.
</pre><br>

  ◎Abrir Archivo (AH=3DH)<br>
<pre>
      Entrada: AL = Modo de acceso
               DS:DX = Nombre de ruta
      Salida:  AX = Manejador de archivo (file handle)
      Explicación: Si el bit 2 del modo de acceso es 1, se abre con «bloqueo» y no se podrá cerrar posteriormente.
</pre><br>

  ◎Obtener/Establecer Atributos de Archivo (AH=43H)<br>
<pre>
      Entrada: AL = 0: Obtener / 1: Establecer
               CX = Atributos
               DS:DX = Nombre de ruta
      Salida:  CX = Atributos
      Explicación: La relación bits-atributos es la siguiente:
                b0 ･････ Solo lectura (Read-only)
                b1 ･････ Invisible
                b2 ･････ Archivo de sistema
                b3 ･････ Reservado
                b4 ･････ Subdirectorio
                b5 ･････ No respaldado (Not backed up)
                b6 ･････ Tipo de archivo (0: Texto, 1: Binario)
                b7 ･････ Reservado
</pre><br>

  ◎Obtener Información de Dispositivo (AX=4400H)<br>
<pre>
      Entrada: AL = 0
               BX = Manejador de archivo
      Salida:  DX = Información del dispositivo
      Explicación: Nótese que en la información devuelta, ISDEV (tipo de dispositivo) está en el bit 8.
</pre><br>

  ◎Obtener Mapa de Unidades (AX=440EH)<br>
<pre>
      Entrada: AL = 0EH
      Salida:  AX = Mapa de unidades
      Explicación: La relación bits-unidad es:
                b15 b14 b13 b12 b11 b10 b9 b8 b7 b6 b5 b4 b3 b2 b1 b0
                E5  E4  E3  E2  E1  E0  F9 F8 F7 F6 F5 F4 F3 F2 F1 --
            El bit correspondiente se pone a 1 si la unidad está disponible. El comportamiento de esta función difiere
            de MS-DOS.
</pre><br>

  ◎Terminar Proceso (AH=4CH)<br>
<pre>
      Entrada: AL = Código de retorno
      Salida:  Ninguna
      Explicación: Vuelve a la línea de comandos de DOS.
</pre><br>


  ◎Formatear Disco (AH=80H)<br>
<pre>
      Entrada: DL = Número de unidad
      Salida:  Ninguna
      Explicación: Ordena al controlador de dispositivo que formatee la unidad. Correspondencia nombre/número de unidad:
                   F1〜F9 E0〜E5
                   1 〜9  10〜15
</pre><br>

  ◎Obtener Cadena de Mensaje (AH=81H)<br>
<pre>
      Entrada: AL = Código de mensaje
               DS:DX = Búfer (64 bytes)
      Salida:  CX = Longitud del mensaje
               Búfer = Cadena de mensaje
      Explicación: La cadena termina con CR, LF. La correspondencia código-cadena se describe más adelante.
</pre><br>

  ◎Establecer Volumen del Click del Teclado (AH=82H)<br>
<pre>
      Entrada: DL = Volumen (0〜255)
      Salida:  Ninguna
      Explicación: Volumen 0 = silencio.
</pre><br>

  ◎Obtener Volumen del Click del Teclado (AH=83H)<br>
<pre>
      Entrada: Ninguna
      Salida:  AL = Volumen
      Explicación: Ninguna.
</pre><br>

  ◎Obtener Tamaño del Área de Máquina (AH=84H)<br>
<pre>
      Entrada: Ninguna
      Salida:  AX = Tamaño del área de máquina (en bytes)
      Explicación: El tamaño se da en bytes.
</pre><br>

  ◎Convertir Nombre de Archivo a Formato Estándar (AH=85H)<br>
<pre>
      Entrada: DS:DX = Nombre de ruta
               ES:BX = Búfer (16 bytes)
      Salida:  Búfer = Nombre en formato estándar (cadena ASCIIZ)
      Explicación: El formato estándar es: nombre de unidad + nombre de archivo (8 caracteres) + ‘.’ + extensión (3
                   caracteres).
</pre><br>

  ◎Convertir Nombre de Archivo a Formato FCB (AH=86H)<br>
<pre>
      Entrada: DS:DX = Nombre de ruta
               ES:BX = Búfer (12 bytes)
      Salida:  Búfer = Nombre en formato FCB
      Explicación: El formato FCB es: número de unidad + nombre de archivo (8 caracteres) + extensión (3 caracteres).
</pre><br>

  ◎Ejecutar Archivo Batch (AH=87H)<br>
<pre>
      Entrada: DS:DX = Nombre de ruta
               ES:BX = Cadena de parámetros (ASCIIZ)
      Salida:  Ninguna
      Explicación: Al terminar la ejecución del archivo batch, el control vuelve a la línea de comandos.
</pre><br>

  ◎Obtener Día de la Semana (AH=88H)<br>
<pre>
      Entrada: CX = Año (1980〜2079)
               DH = Mes (1〜12)
               DL = Día (1〜31)
      Salida:  AL = Día de la semana (0:Dom, 1:Lun, 2:Mar, 3:Mié, 4:Jue, 5:Vie, 6:Sáb)
      Explicación: Calculado basándose en el calendario pitagórico.
</pre><br>

  ◎Registrar Variable de Entorno (AH=89H)<br>
<pre>
      Entrada: DS:DX = Nombre de variable
               ES:BX = Contenido
      Salida:  Ninguna
      Explicación: El nombre de variable y el contenido deben terminar con 00H. Si el contenido es solo 00H, se elimina
                   la variable.
</pre><br>

  ◎Obtener Variable de Entorno (AH=8AH)<br>
<pre>
      Entrada: DS:DX = Nombre de variable
               ES:BX = Búfer
      Salida:  AX = Longitud del contenido
               Búfer = Contenido
      Explicación: El nombre de variable debe terminar con 00H. El contenido de las variables de entorno no tiene
                   límite de tamaño, así que asigne el búfer lo más grande posible.
</pre><br>

  ◎Asignar FFE (AH=8BH)<br>
<pre>
      Entrada: BX = Tamaño (en párrafos)
      Salida:  AX = Segmento inicial
      Explicación: Obtiene un área de trabajo desde el Área Libre de Archivos (File Free Area). El área asignada no se
                   libera hasta apagar el equipo.
</pre><br>

  ◎Especificar Búfer Histórico para CON (AH=8CH)<br>
<pre>
      Entrada: DS:DX = Búfer histórico (258 bytes)
      Salida:  Ninguna
      Explicación: Especifica el búfer histórico para el dispositivo «CON». Se usa para cambiar el historial cuando se
                   usa entrada de teclado con búfer (AH=0AH).
</pre><br>

  ◎Abrir AUX (AH=8DH)<br>
<pre>
      Entrada: Ninguna
      Salida:  Ninguna
      Explicación: Abre directamente el dispositivo AUX. Los datos recibidos por el puerto serial se escriben en
                   «RXCNT» del área del sistema (devuelto en DI por la BIOS en B9H), desde donde se pueden obtener.
</pre><br>

  ◎Cerrar AUX (AH=8EH)<br>
<pre>
      Entrada: Ninguna
      Salida:  Ninguna
      Explicación: Cierra el dispositivo AUX abierto con la función anterior. Si AUX se abrió múltiples veces, se debe
                   cerrar la misma cantidad de veces para que se cierre correctamente.
</pre><br>

  ◎Analizar Ruta de Directorio (AH=8FH)<br>
<pre>
      Entrada: DS:DX = Cadena con nombre de archivo
               ES:BX = Búfer (64 bytes)
      Salida:  Búfer = Ruta de directorio
      Explicación: Extrae la parte correspondiente al directorio de la cadena de nombre de archivo. Si se omitió el
                   directorio, se deduce la ruta a partir del directorio actual.
</pre><br>


## ３．Cadenas de Mensaje

　La correspondencia entre los códigos de mensaje y las cadenas obtenibles con la función 81H es la siguiente:<br><br>
<pre>
         0:???
         1:Cannot format this drive
         2:Unrecognized command
         3:Command too long
         4:File cannot be copied onto itself
         5:Disk full
         6:Too many parameters
         7:Missing parameter
         8:File not found
         9:Read only file
        10:Cannot overwrite previous destination file
        11:Duplicate filename
        12:Invalid filename
        13:Invalid option
        14:Invalid environment string
        15:Invalid time
        16:Invalid number
        17:Invalid drive
        18:Invalid parameter
        19:Invalid date
        20:Not enough memory
        21:No spare file handles
        22:Cannot access
        23:Out of system free
        24:Bath file error
        25:Strike a key when ready
        26:Invaild 'EXE' format
        27:Buffer overflow
        28:Framing error
        29:Overrun error
        30:Parity error
        31:Disk write-protected
        32:Disk is not ready
        33:Path not found
</pre><br>


## ４．Puntos a Considerar al Crear Programas

<pre>
  ◎Asignación de Memoria
    　La memoria asignada mediante la función 48H se reserva entre el final del sistema DOS y el área de archivos
       «F0» y «F1», y se libera para el usuario. Sin embargo, esta asignación es solo temporal y se elimina al volver
       a la línea de comandos.
    　Por lo tanto, no se puede usar para registrar programas residentes como en MS-DOS.
    　Para registrar programas residentes, use la función 8BH (Asignar FFE).
    　FFE (Área Libre de Archivos) es el espacio entre el área de archivos «F9» y el final de la RAM. La asignación
       toma la cantidad necesaria desde el final de esta área.

  ◎Acceso a Archivos
    　Al acceder a la unidad básica, escribir en un archivo y expandir su tamaño lleva mucho tiempo. Expandir
       previamente el archivo al tamaño deseado antes de escribir los datos puede acelerar el procesamiento.

  ◎Máximo de Manejadores de Archivo
    　El número máximo de archivos (incluyendo dispositivos) que se pueden abrir simultáneamente es 20.

  ◎Uso de la BIOS
    　Se permite usar la BIOS en programas para FX-DOS, pero no la use para mostrar caracteres.
    　Esto es para poder aprovechar futuros controladores de visualización de texto.
    　Además, preste atención a los segmentos al usar la BIOS. Algunas rutinas no funcionan correctamente si DS no es 0.

  ◎Entrada de Teclado
    　Hay dos métodos para entrada de teclado: usar la BIOS o usar INT 2BH. Es mejor usarlos según la situación.
    　Por ejemplo, usar INT 2BH al mover el cursor podría activar el FEP innecesariamente.
    　Las funciones 03H y 04H de la BIOS no permiten autorepetición (key repeat) por sí solas, pero ejecutando la
       función 47H de la BIOS justo después se puede habilitar.

  ◎Puntero de Pila (Stack Pointer)
    　Básicamente, no cambie el valor del puntero de pila (SP).
    　La pila durante la ejecución de una aplicación es la pila del sistema (esto aplica a formatos EXE, COM/CMD y
       BIN). Si la mueve a otro segmento, la entrada de teclado dejará de funcionar debido a limitaciones de la BIOS.
    　Al ejecutar la terminación de la aplicación (función 4CH), no es necesario restaurar el nivel de la pila.

  ◎Manejo de la Instrucción 'ESC' en Nemónicos
    　Ejecutar la instrucción nemotécnica 'ESC' en el CPU 80188EB de la FX-890P causa un bloqueo. FX-DOS incluye
       contramedidas para evitar este bloqueo.
    　Esto permite ejecutar software de MS-DOS en FX-DOS, ya que algunos programas de MS-DOS usan esta instrucción.

  ◎Sobre Software para MS-DOS
    　FX-DOS procesa los formatos de archivo COM y EXE exactamente igual que MS-DOS.
    　Por lo tanto, es posible crear software para FX-DOS usando lenguajes de alto nivel como C.
</pre>
