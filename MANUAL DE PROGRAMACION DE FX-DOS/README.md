# Manual del Programador de FX-DOS

## Capítulo de Solicitudes de Función

　Este documento explica la información necesaria al crear programas para FX-DOS, especialmente lo relacionado con las solicitudes de función (function requests).<br>
　Se asume que el lector posee un conocimiento suficiente de MS-DOS. Se recomienda a quienes no lo tengan que preparen material de referencia sobre MS-DOS por separado.<br><br>

## １．Llamadas al Sistema

 Cuando FX-DOS inicia en frío (cold start), varias interrupciones de software (software interrupts) quedan disponibles. Estas interrupciones de software se denominan llamadas al sistema (system calls), y generalmente son utilizadas por las aplicaciones para acceder a las funcionalidades de FX-DOS.<br><br>
        INT 21H ･･･ Solicitud de Función (Function Request)<br>
        INT 23H ･･･ Dirección de Escape con [BRK]<br>
        INT 25H ･･･ Lectura Absoluta de Disco (Absolute Disk Read)<br>
        INT 26H ･･･ Escritura Absoluta de Disco (Absolute Disk Write)<br>
        INT 29H ･･･ Salida Directa a Consola (Direct Console Output)<br>
        INT 2BH ･･･ Entrada del FEP<br>
        INT 2CH ･･･ Acceso Directo a Archivos (Direct File Access)<br>
        INT 30H ･･･ Información de Estado del FEP<br><br>

  ◎Solicitud de Función (INT 21H)<br>
      Entrada: AH = Número de función<br>
               Otros según la función llamada.<br>
      Salida: Según la función llamada.<br>
      Registros Afectados: AX<br>
      Explicación: Proporciona la mayoría de las funcionalidades ofrecidas por FX-DOS, como operaciones de archivos y entrada/salida de datos.<br><br>

  ◎Dirección de Escape con [BRK] (INT 23H)<br>
      Entrada: Ninguna<br>
      Salida: Ninguna<br>
      Explicación: Especifica la dirección de la rutina ejecutada al presionar [BRK].<br>
                   Por defecto, cierra todos los archivos y vuelve a la línea de comandos, pero el usuario puede proporcionar su propia rutina de manejo.<br>
                   Si el usuario la maneja, debe restablecer SP dentro de la rutina.<br><br>

  ◎Lectura/Escritura Absoluta de Disco (INT 25H/26H)<br>
      Entrada: AL = Número de unidad (E0=10, E1=11, ..., E5=15)<br>
               DS:BX = Dirección de transferencia de datos<br>
               CX = Número de sectores<br>
               DX = Número de sector inicial<br>
      Salida: Cy, AL = Código de error<br>
      Registros Afectados: AX, BX, CX, DX, BP, DI, SI<br>
      Explicación: Intercambia datos con unidades extendidas por sectores.<br>
                   Esta función solo puede usarse con unidades extendidas y solo si el controlador de dispositivo la soporta.<br>
                   Atención: Los flags al momento de la interrupción quedan en la pila al retornar, por lo que se debe usar «POP F» para restaurar el nivel de la pila.<br><br>

  ◎Salida Directa a Consola (INT 29H)<br>
      Entrada: AL = Carácter a mostrar<br>
      Salida: Ninguna<br>
      Registros Afectados: Ninguno<br>
      Explicación: Envía datos directamente a la rutina de visualización de caracteres.<br>
                   Se conecta directamente al controlador de visualización de texto, permitiendo una salida muy rápida.<br><br>

  ◎Entrada del FEP (INT 2BH)<br>
      Entrada: Ninguna<br>
      Salida: AL = Carácter ingresado<br>
              AH = Número de caracteres acumulados en el buffer<br>
      Registros Afectados: AX<br>
      Explicación: Lee datos desde el controlador de entrada de teclado (se puede usar incluso sin FEP).<br>
                   Espera si no hay datos de entrada.<br><br>

  ◎Acceso Directo a Archivos (INT 2CH)<br>
      Entrada: DS:DX = Nombre de archivo en formato FCB<br>
      Salida: DX:AX = Tamaño del archivo<br>
              DS:SI = Dirección inicial del archivo<br>
              Cy, AX = Código de error (2, 5, 15)<br>
      Registros Afectados: AX, BX, CX, DX, BP, DI, SI, DS, ES<br>
      Explicación: Obtiene la dirección de inicio de la parte de datos de un archivo en la unidad básica.<br>
                   El contenido del archivo reside en un espacio de memoria contiguo comenzando en DS:SI.<br>
                   Se usa para acceder a archivos a alta velocidad, pero su uso indebido puede dañar otros archivos. Úselo con precaución.<br>
                   ※ Si se accede al archivo mediante otras funciones después de obtener su dirección inicial, esta información pierde validez.<br><br>

  ◎Información de Estado del FEP (INT 30H)<br>
      Entrada: AH = 0 Obtener versión (AX)<br>
                   1 Obtener modo de entrada (AL)<br>
                   2 Establecer modo de entrada (AL=0: ANK, FF: Kanji)<br>
      Salida: AX = Versión (si entrada AH=0)<br>
              AL = Modo de entrada (0: ANK, FF: Kanji) (si entrada AH=1)<br>
      Registros Afectados: AX<br>
      Explicación: Consulta o configura el estado del FEP (Front End Processor).<br>
                   No se puede usar si no hay FEP, pero esto permite verificar si un FEP está registrado.<br>
                   Si se ejecuta INT 30H con AX=0 y AX sigue siendo 0, no hay FEP registrado.<br><br>


## ２．Solicitudes de Función (Function Requests)

　La mayor parte de los servicios proporcionados por FX-DOS están disponibles a través de estas solicitudes de función.<br>
　Están basadas fundamentalmente en las funciones de MS-DOS, pero incluyen algunas con comportamiento ligeramente diferente y otras exclusivas de FX-DOS.<br>
　Tenga en cuenta que los códigos de error devueltos ante problemas pueden diferir de los de MS-DOS.<br><br>

  ※ Nota: Básicamente, AX se modifica en estas funciones.<br>
  ※ Las funciones marcadas con «*» tienen diferencias de comportamiento respecto a MS-DOS.<br><br>

      00H ･･･ Terminar Proceso             3AH ････ Eliminar Subdirectorio<br>
      01H ･･･ Entrada de Tecla (con eco)   3BH ････ Cambiar Directorio Actual<br>
      02H ･･･ Salida a Pantalla            3CH ････ Crear Archivo<br>
      03H ･･･ Salida a AUX                *3DH ････ Abrir Archivo<br>
      04H ･･･ Entrada desde AUX            3EH ････ Cerrar Archivo<br>
      05H ･･･ Salida a PRN                 3FH ････ Leer Archivo<br>
      06H ･･･ E/S Directa de Consola       40H ････ Escribir Archivo<br>
      07H ･･･ Entrada Directa de Consola   41H ････ Eliminar Archivo<br>
      08H ･･･ Entrada de Tecla             42H ････ Mover Puntero de Archivo<br>
      09H ･･･ Salida de Cadena            *43H ････ Obtener/Establecer Atributos<br>
      0AH ･･･ Entrada de Cadena           *4400H ･･ Obtener Info. de Dispositivo<br>
      0BH ･･･ Verificar Teclado            4406H ･･ Obtener Estado de Entrada<br>
      0CH ･･･ Vaciar Buffer y Entrar Tecla 4407H ･･ Obtener Estado de Salida<br>
      0EH ･･･ Establecer Unidad Actual   *440EH ･･ Obtener Mapa de Unidades<br>
      19H ･･･ Obtener Unidad Actual        47H ････ Obtener Directorio Actual<br>
      1AH ･･･ Establecer Dirección DTA     48H ････ Asignar Bloque de Memoria<br>
     *25H ･･･ Establecer Vector Int.       4AH ････ Cambiar Tamaño Bloque Mem.<br>
      2AH ･･･ Obtener Fecha                4BH ････ Ejecutar Programa<br>
      2BH ･･･ Establecer Fecha            *4CH ････ Terminar Proceso<br>
      2CH ･･･ Obtener Hora                 4DH ････ Obtener Código de Salida<br>
      2DH ･･･ Establecer Hora              4EH ････ Buscar Archivo<br>
      2FH ･･･ Obtener Dirección DTA        4FH ････ Continuar Búsqueda Archivo<br>
     *30H ･･･ Obtener Número de Versión    52H ････ Obtener Dirección Var. Entorno<br>
     *31H ･･･ Terminar y Residentar        *56H ････ Renombrar Archivo<br>
     *35H ･･･ Obtener Vector de Interrup.  57H ････ Obtener/Establecer Fecha/Hora<br>
      36H ･･･ Obtener Espacio Libre en Disco5BH ････ Crear Archivo Nuevo<br>
      39H ･･･ Crear Subdirectorio<br>
<br>
     *80H ･･･ Formatear Disco             *88H ･･･ Obtener Día de la Semana<br>
     *81H ･･･ Obtener Cadena de Mensaje   *89H ･･･ Registrar Variable de Entorno<br>
     *82H ･･･ Establecer Volumen Click Tecl*8AH ･･･ Obtener Variable de Entorno<br>
     *83H ･･･ Obtener Volumen Click Tecl  *8BH ･･･ Asignar FFE<br>
     *84H ･･･ Obtener Tamaño Área Máquina *8CH ･･･ Especificar Búfer Histórico CON<br>
     *85H ･･･ Nombre Archivo a Formato Est.*8DH ･･･ Abrir AUX<br>
     *86H ･･･ Nombre Archivo a Formato FCB *8EH ･･･ Cerrar AUX<br>
     *87H ･･･ Ejecutar Archivo Batch      *8FH ･･･ Analizar Ruta de Directorio<br><br>

  ◎Establecer Vector de Interrupción (AH=25H)<br>
      Entrada: AL = Número de interrupción<br>
               DS:DX = Vector<br>
      Salida: Ninguna<br>
      Explicación: Si la dirección del vector es mayor que la dirección inicial del área de archivos «F0» y menor que el final de la RAM, se procesa mediante relocalización automática (self-relocation).<br><br>

  ◎Obtener Número de Versión (AH=30H)<br>
      Entrada: Ninguna<br>
      Salida: AX = 0A03H<br>
              BX = 0FF00H<br>
              CL = Parte entera de la versión<br>
              CH = Parte decimal de la versión<br>
      Explicación: Para la versión 1.23, devuelve «CL=1, CH=23».<br>
                   En MS-DOS, CX devolvería «0» como número de usuario. Verificando esto se puede distinguir si el SO es FX-DOS o MS-DOS.<br><br>

  ◎Terminar y Dejar Residente (AH=31H)<br>
      Entrada: AL = Código de salida<br>
      Salida: Ninguna<br>
      Explicación: Existe por compatibilidad con MS-DOS. Internamente funciona exactamente igual que «Terminar Proceso (AH=4CH)».<br><br>

  ◎Obtener Vector de Interrupción (AH=35H)<br>
      Entrada: AL = Número de interrupción<br>
      Salida: AL = Flag de enlace (0FFH: Enlace DOS)<br>
              ES:BX = Vector<br>
      Explicación: «Enlace DOS» indica que la rutina de manejo de la interrupción reside dentro del sistema DOS.<br><br>

  ◎Abrir Archivo (AH=3DH)<br>
      Entrada: AL = Modo de acceso<br>
               DS:DX = Nombre de ruta<br>
      Salida: AX = Manejador de archivo (file handle)<br>
      Explicación: Si el bit 2 del modo de acceso es 1, se abre con «bloqueo» y no se podrá cerrar posteriormente.<br><br>

  ◎Obtener/Establecer Atributos de Archivo (AH=43H)<br>
      Entrada: AL = 0: Obtener / 1: Establecer<br>
               CX = Atributos<br>
               DS:DX = Nombre de ruta<br>
      Salida: CX = Atributos<br>
      Explicación: La relación bits-atributos es la siguiente:<br>
                b0 ･････ Solo lectura (Read-only)<br>
                b1 ･････ Invisible<br>
                b2 ･････ Archivo de sistema<br>
                b3 ･････ Reservado<br>
                b4 ･････ Subdirectorio<br>
                b5 ･････ No respaldado (Not backed up)<br>
                b6 ･････ Tipo de archivo (0: Texto, 1: Binario)<br>
                b7 ･････ Reservado<br><br>

  ◎Obtener Información de Dispositivo (AX=4400H)<br>
      Entrada: AL = 0<br>
               BX = Manejador de archivo<br>
      Salida: DX = Información del dispositivo<br>
      Explicación: Nótese que en la información devuelta, ISDEV (tipo de dispositivo) está en el bit 8.<br><br>

  ◎Obtener Mapa de Unidades (AX=440EH)<br>
      Entrada: AL = 0EH<br>
      Salida: AX = Mapa de unidades<br>
      Explicación: La relación bits-unidad es:<br>
                b15 b14 b13 b12 b11 b10 b9 b8 b7 b6 b5 b4 b3 b2 b1 b0<br>
                E5  E4  E3  E2  E1  E0  F9 F8 F7 F6 F5 F4 F3 F2 F1 --<br>
            El bit correspondiente se pone a 1 si la unidad está disponible.<br>
            El comportamiento de esta función difiere de MS-DOS.<br><br>

  ◎Terminar Proceso (AH=4CH)<br>
      Entrada: AL = Código de retorno<br>
      Salida: Ninguna<br>
      Explicación: Vuelve a la línea de comandos de DOS.<br><br>


  ◎Formatear Disco (AH=80H)<br>
      Entrada: DL = Número de unidad<br>
      Salida: Ninguna<br>
      Explicación: Ordena al controlador de dispositivo que formatee la unidad.<br>
                   Correspondencia nombre/número de unidad:<br>
                   F1〜F9 E0〜E5<br>
                   1 〜9  10〜15<br><br>

  ◎Obtener Cadena de Mensaje (AH=81H)<br>
      Entrada: AL = Código de mensaje<br>
               DS:DX = Búfer (64 bytes)<br>
      Salida: CX = Longitud del mensaje<br>
              Búfer = Cadena de mensaje<br>
      Explicación: La cadena termina con CR, LF.<br>
                   La correspondencia código-cadena se describe más adelante.<br><br>

  ◎Establecer Volumen del Click del Teclado (AH=82H)<br>
      Entrada: DL = Volumen (0〜255)<br>
      Salida: Ninguna<br>
      Explicación: Volumen 0 = silencio.<br><br>

  ◎Obtener Volumen del Click del Teclado (AH=83H)<br>
      Entrada: Ninguna<br>
      Salida: AL = Volumen<br>
      Explicación: Ninguna.<br><br>

  ◎Obtener Tamaño del Área de Máquina (AH=84H)<br>
      Entrada: Ninguna<br>
      Salida: AX = Tamaño del área de máquina (en bytes)<br>
      Explicación: El tamaño se da en bytes.<br><br>

  ◎Convertir Nombre de Archivo a Formato Estándar (AH=85H)<br>
      Entrada: DS:DX = Nombre de ruta<br>
               ES:BX = Búfer (16 bytes)<br>
      Salida: Búfer = Nombre en formato estándar (cadena ASCIIZ)<br>
      Explicación: El formato estándar es: nombre de unidad + nombre de archivo (8 caracteres) + ‘.’ + extensión (3 caracteres).<br>
                   Ej: «F1:CONFIG  .SYS¥0»<br><br>

  ◎Convertir Nombre de Archivo a Formato FCB (AH=86H)<br>
      Entrada: DS:DX = Nombre de ruta<br>
               ES:BX = Búfer (12 bytes)<br>
      Salida: Búfer = Nombre en formato FCB<br>
      Explicación: El formato FCB es: número de unidad + nombre de archivo (8 caracteres) + extensión (3 caracteres).<br>
                   Ej: «¥1CONFIG  SYS»<br><br>

  ◎Ejecutar Archivo Batch (AH=87H)<br>
      Entrada: DS:DX = Nombre de ruta<br>
               ES:BX = Cadena de parámetros (ASCIIZ)<br>
      Salida: Ninguna<br>
      Explicación: Al terminar la ejecución del archivo batch, el control vuelve a la línea de comandos.<br><br>

  ◎Obtener Día de la Semana (AH=88H)<br>
      Entrada: CX = Año (1980〜2079)<br>
               DH = Mes (1〜12)<br>
               DL = Día (1〜31)<br>
      Salida: AL = Día de la semana (0:Dom, 1:Lun, 2:Mar, 3:Mié, 4:Jue, 5:Vie, 6:Sáb)<br>
      Explicación: Calculado basándose en el calendario pitagórico.<br><br>

  ◎Registrar Variable de Entorno (AH=89H)<br>
      Entrada: DS:DX = Nombre de variable<br>
               ES:BX = Contenido<br>
      Salida: Ninguna<br>
      Explicación: El nombre de variable y el contenido deben terminar con 00H.<br>
                   Si el contenido es solo 00H, se elimina la variable.<br>
                   Si ya existe una variable con el mismo nombre, se elimina la anterior antes de registrar la nueva.<br><br>

  ◎Obtener Variable de Entorno (AH=8AH)<br>
      Entrada: DS:DX = Nombre de variable<br>
               ES:BX = Búfer<br>
      Salida: AX = Longitud del contenido<br>
              Búfer = Contenido<br>
      Explicación: El nombre de variable debe terminar con 00H.<br>
                   El contenido de las variables de entorno no tiene límite de tamaño, así que asigne el búfer lo más grande posible.<br><br>

  ◎Asignar FFE (AH=8BH)<br>
      Entrada: BX = Tamaño (en párrafos)<br>
      Salida: AX = Segmento inicial<br>
      Explicación: Obtiene un área de trabajo desde el Área Libre de Archivos (File Free Area).<br>
                   El área asignada no se libera hasta apagar el equipo.<br>
                   Se usa principalmente para que los controladores de dispositivo obtengan un área residente.<br><br>

  ◎Especificar Búfer Histórico para CON (AH=8CH)<br>
      Entrada: DS:DX = Búfer histórico (258 bytes)<br>
      Salida: Ninguna<br>
      Explicación: Especifica el búfer histórico para el dispositivo «CON».<br>
                   Se usa para cambiar el historial cuando se usa entrada de teclado con búfer (AH=0AH).<br>
                   Si DX = FFFFH, se usa el predeterminado (línea de comandos de DOS).<br>
                   Formato del búfer histórico:<br>
                   Posición de lectura (1 byte), Posición de escritura (1 byte), Búfer (256 bytes)<br>
                   El búfer es circular, y cada línea termina en «[CR]¥0».<br><br>

  ◎Abrir AUX (AH=8DH)<br>
      Entrada: Ninguna<br>
      Salida: Ninguna<br>
      Explicación: Abre directamente el dispositivo AUX.<br>
                   Los datos recibidos por el puerto serial se escriben en «RXCNT» del área del sistema (devuelto en DI por la BIOS en B9H), desde donde se pueden obtener.<br><br>

  ◎Cerrar AUX (AH=8EH)<br>
      Entrada: Ninguna<br>
      Salida: Ninguna<br>
      Explicación: Cierra el dispositivo AUX abierto con la función anterior.<br>
                   Si AUX se abrió múltiples veces, se debe cerrar la misma cantidad de veces para que se cierre correctamente. (No hay problema si se cierra de más).<br><br>

  ◎Analizar Ruta de Directorio (AH=8FH)<br>
      Entrada: DS:DX = Cadena con nombre de archivo<br>
               ES:BX = Búfer (64 bytes)<br>
      Salida: Búfer = Ruta de directorio<br>
      Explicación: Extrae la parte correspondiente al directorio de la cadena de nombre de archivo.<br>
                   Si se omitió el directorio, se deduce la ruta a partir del directorio actual.<br>
                   Ej: «F1:¥¥0»<br>
                       «E1:¥DATA¥ETC¥¥0»<br><br>


## ３．Cadenas de Mensaje

　La correspondencia entre los códigos de mensaje y las cadenas obtenibles con la función 81H es la siguiente:<br><br>
         0:???<br>
         1:Cannot format this drive<br>
         2:Unrecognized command<br>
         3:Command too long<br>
         4:File cannot be copied onto itself<br>
         5:Disk full<br>
         6:Too many parameters<br>
         7:Missing parameter<br>
         8:File not found<br>
         9:Read only file<br>
        10:Cannot overwrite previous destination file<br>
        11:Duplicate filename<br>
        12:Invalid filename<br>
        13:Invalid option<br>
        14:Invalid environment string<br>
        15:Invalid time<br>
        16:Invalid number<br>
        17:Invalid drive<br>
        18:Invalid parameter<br>
        19:Invalid date<br>
        20:Not enough memory<br>
        21:No spare file handles<br>
        22:Cannot access<br>
        23:Out of system free<br>
        24:Bath file error<br>
        25:Strike a key when ready<br>
        26:Invaild 'EXE' format<br>
        27:Buffer overflow<br>
        28:Framing error<br>
        29:Overrun error<br>
        30:Parity error<br>
        31:Disk write-protected<br>
        32:Disk is not ready<br>
        33:Path not found<br><br>


## ４．Puntos a Considerar al Crear Programas

  ◎Asignación de Memoria<br>
    　La memoria asignada mediante la función 48H se reserva entre el final del sistema DOS y el área de archivos «F0» y «F1», y se libera para el usuario. Sin embargo, esta asignación es solo temporal y se elimina al volver a la línea de comandos.<br>
    　Por lo tanto, no se puede usar para registrar programas residentes como en MS-DOS.<br>
    　Para registrar programas residentes, use la función 8BH (Asignar FFE).<br>
    　FFE (Área Libre de Archivos) es el espacio entre el área de archivos «F9» y el final de la RAM. La asignación toma la cantidad necesaria desde el final de esta área.<br><br>

  ◎Acceso a Archivos<br>
    　Al acceder a la unidad básica, escribir en un archivo y expandir su tamaño lleva mucho tiempo. Expandir previamente el archivo al tamaño deseado antes de escribir los datos puede acelerar el procesamiento.<br><br>

  ◎Máximo de Manejadores de Archivo<br>
    　El número máximo de archivos (incluyendo dispositivos) que se pueden abrir simultáneamente es 20.<br><br>

  ◎Uso de la BIOS<br>
    　Se permite usar la BIOS en programas para FX-DOS, pero no la use para mostrar caracteres.<br>
    　Esto es para poder aprovechar futuros controladores de visualización de texto.<br>
    　Además, preste atención a los segmentos al usar la BIOS. Algunas rutinas no funcionan correctamente si DS no es 0.<br><br>

  ◎Entrada de Teclado<br>
    　Hay dos métodos para entrada de teclado: usar la BIOS o usar INT 2BH. Es mejor usarlos según la situación.<br>
    　Por ejemplo, usar INT 2BH al mover el cursor podría activar el FEP innecesariamente.<br>
    　Las funciones 03H y 04H de la BIOS no permiten autorepetición (key repeat) por sí solas, pero ejecutando la función 47H de la BIOS justo después se puede habilitar.<br><br>

  ◎Puntero de Pila (Stack Pointer)<br>
    　Básicamente, no cambie el valor del puntero de pila (SP).<br>
    　La pila durante la ejecución de una aplicación es la pila del sistema (esto aplica a formatos EXE, COM/CMD y BIN). Si la mueve a otro segmento, la entrada de teclado dejará de funcionar debido a limitaciones de la BIOS.<br>
    　Al ejecutar la terminación de la aplicación (función 4CH), no es necesario restaurar el nivel de la pila.<br><br>

  ◎Manejo de la Instrucción 'ESC' en Nemónicos<br>
    　Ejecutar la instrucción nemotécnica 'ESC' en el CPU 80188EB de la FX-890P causa un bloqueo. FX-DOS incluye contramedidas para evitar este bloqueo.<br>
    　Esto permite ejecutar software de MS-DOS en FX-DOS, ya que algunos programas de MS-DOS usan esta instrucción.<br><br>

  ◎Sobre Software para MS-DOS<br>
    　FX-DOS procesa los formatos de archivo COM y EXE exactamente igual que MS-DOS.<br>
    　Por lo tanto, es posible crear software para FX-DOS usando lenguajes de alto nivel como C.

