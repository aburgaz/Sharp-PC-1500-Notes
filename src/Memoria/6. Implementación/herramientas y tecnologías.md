### **Herramientas**

La comunidad de emulación y recreación de ordenadores antiguos aportan guías para usuarios nuevos en las que detallan paso a paso el proceso de creación de emuladores. De igual manera abundan los resursos didácticos destinados a la codificación de compiladores modernos. Sin embargo no se especifica herramientas concretas para realizar tales tareas.

#### **Herramientas software**

**Rust**: como lenguaje de programación es altamente eficiente, gracias a eso tanto el emulador como el compilador pueden ejecutarse a gran velocidad y mejorar la experiencia de usuario. Además es seguro en memoria gracias a su sistema de *ownership* y *borrowing*. El porpio lenguaje obliga a manejar correctamente los errores y cuenta con una gran comunidad activa y abundant documentación, soporte y librerías actualizadas.

**Haskell**: se trata de un lenguaje de programación funcional de proposito general creado en 1990 con el fin de estandarizar la este paradigma. Es fuertemente tipado y se caracteriza por la llamada *lazy evaluation*, que consiste en calcular las expresiones solo en el momento en el que su valor es realmente necesario. Resulta de utilidad para trabajar con estructuras de datos infinitas, evitar calculos innecesarios y puede mejorar la eficiencia casos especificos. 

**BASIC**: uno de los primeros lenguajes de alto nivel que cuenta con gran cantidad de dialectos diferentes. Este se volvió muy popular en una epoca donde el software se hacía a medida y sus variantes resultaron muy útiles en el mundo de los microcomputadoras. Es el lenguaje soportado por el PC-1500, todos los programas ejecutados por la calculadora están escritos en BASIC.

**EGUI**: se trata de una biblioteca de interfaz gráfica de usuario para Rust creada por el desarrollador Emil Ernerfeldt. Se conoce por ser fácil de usar y ligera, además de ser portable a proyectos web. Gracias a su simplicidad permite crear interfaces de usuario rápidamente y muy eficientes, ideal para renderizar las vistas del simulador. Rust no es un lenguaje que esté actualmente enfocado en la creación de aplicaciones de escritorio, sin embargo la comunidad aporta grandes ideas y proyectos de código abierto para ampliar sus capacidades.

**CoolTerm**: Un software libre construido para comunicar sistemas a traves de sus puertos serie, ya sea usb o Bluetooth através de una terminal. Creado por Roger Meier para Windows, es sencillo de usar y conveniente para comunicarse a través de un conector con piezas de hardware. El creador de la interfaz para el PC-1500, jeff Birt, usa este programa para extraer la ROM de la máquina utilizando la terminal integrada.

**TeraTerm**: Similar al software anterior está dedicado a la comunicación de equipos a traves de conectores en serie o protocolos de red. Presenta una interfaz sencilla, una terminal y muy pocas funcionalidades.

**PockEmul**: un blog dedicado al mantenimiento de las computadoras de bolsillo programables. Se encarga de ofrecer la documentación tecnica y manuales de una varidad de máquinas de la época de los 70 a los 90, e informar de las novedades que puedan resultar de interés para la comunidad. Además ofrece un simulador con un catálogo de ordenadores de bolsillo para probar y programar en ellos. Disponible tanto como aplicación de escritorio como una versión en el navegador web.

**LH Tools/LHasm**: un conjunto de herramientas útiles diseñado especificamente para la SHARP PC-1500. COnsiste en 4 programas diferentes: *lhasm*,*lhdump*, *lhcom* y *lhpoke*. El primero, **lhasm**, resulta de gran ayuda para generar binarios de porgramas en BASIC siguiendo la gramática descrita en la ROM de la calculadora de bolsillo y de esta manera comparar los resultados con los obtenidos por el compilador descrito aquí.



#### **Herramientas hardware**

**Interfaz en serie CE-158X**: Su creador, Jeff Birt, la describe como una interfaz en paralelo/serie especifica para el PC-1500 de SHARP y PC-1600. CReada para suplir la escasez de interfaces econimicas y veloces del mercado. Con una velocidad de trensferencia de 19200 bps o inculuso 38400 bps en entornos experimentales es ideal para tranferir programas al dispositivo rápidamente. Además añade un segundo puerto en serie USB que facilita todavía más la comunicación con sistemas actuales. Esta interfaz se conceta al PC-1500 a través del bus de salida lateral normalmente.

**SHARP PC-1500**: La propia calculadora programable de bolsillo alimentada con pilas resulta una herramienta esencial a la hora de probar el código generado por el compilador. Unida a través del bus de datos en su costado a la interfaz CE-158X se realizan treanferencias bidireccionales de datos. Las funcionalidades del simulador han sido pulidas gracias a la posesión de esta reliquia.