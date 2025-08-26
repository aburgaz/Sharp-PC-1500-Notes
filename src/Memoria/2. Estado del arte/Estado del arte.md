## Estado del arte

### 1. Emulación del SHARP PC‑1500

La emulación de ordenadores de bolsillo SHARP ha avanzado en los últimos años gracias a proyectos multiplaforma. PockEmul es actualmente la referencia más activa: soporta emuladores, compiladores y desensambladores para PC‑1500/PC‑1500A, integra periféricos y mantiene actualizaciones que corrigen ROMs y compatibilidad, lo que lo convierte en una base consolidada para validación funcional y preservación. 
Además, existen emuladores históricos dedicados a Windows, como el PC‑1500A Emulator de Philippe Dupas (Win9x–XP). Aunque desactualizado, ilustra enfoques clásicos de emulación “monolítica” centrados en CPU + LCD + teclado con carga de imágenes de cinta. En la web también se encuentran iniciativas modernas en JavaScript/TypeScript (p. ej., Forever1500) que priorizan la accesibilidad en navegador, útiles para docencia y difusión. 
El ecosistema general de emulación (MAME/MESS) ha dado cobertura desigual a “pocket computers”; la documentación comunitaria lo recoge, pero no existe una cadena de herramientas integral orientada a desarrollo contemporáneo (compilar‑ejecutar‑probar) específicamente para PC‑1500. Esta fragmentación refuerza la oportunidad de integrar emulación precisa con tooling de desarrollo.

### 2. Herramientas de desarrollo (ensambladores, compiladores, utilidades)

Históricamente, la programación del PC‑1500 se ha realizado en BASIC y código máquina sobre el microprocesador Sharp LH5801. La comunidad ha producido utilidades como desensambladores, recursos de ensamblador y, de forma puntual, menciones a compiladores de alto nivel; sin embargo, estos esfuerzos están dispersos, con escasa documentación unificada y cobertura parcial de la arquitectura y del conjunto de periféricos. 
En contraste con plataformas más populares de 8 bits, no existe un toolchain moderno ampliamente adoptado (lexer/parser, IR, backend a LH5801) que permita escribir en un lenguaje de alto nivel y generar binarios ejecutables o listados POKE/LOAD listos para el hardware o para emuladores actuales. Esta ausencia de un flujo “moderno” (compilación reproducible, pruebas automatizadas, integración con emulador) constituye un vacío claro que este trabajo aborda.

### 3. Documentación técnica y manuales

La base primaria para comprender el sistema la constituyen los manuales oficiales:

- Technical Reference Manual del PC‑1500, que detalla la CPU LH5801, mapa de memoria, puertos de E/S y rutinas de sistema; es la referencia para especificar la semántica de instrucciones y los detalles de temporización y periféricos.
- Manual de usuario/Programación en BASIC del PC‑1500/PC‑1500A, clave para compatibilidad de intérprete, formatos de cinta y convenciones de sistema. 

Recursos comunitarios como PC‑1500.info agregan notas sobre la LH5801, enlaces a libros y utilidades, resultando valiosos para resolver ambigüedades y recopilar variantes de ROM y módulos. Para una panorámica general de la plataforma (fechas, modelos derivados, relación con TRS‑80 PC‑2), la entrada enciclopédica es útil como punto de partida.

### 4. Comunidad y proyectos relacionados

Blogs técnicos y repositorios públicos muestran un interés sostenido por el PC‑1500 y, en particular, por la LH5801 (p. ej., diseños de PCB, notas de ingeniería inversa, colecciones de ROM). Este corpus informal complementa los manuales y alimenta el conocimiento práctico necesario para una emulación cíclica exacta y para la generación de código. Por su naturaleza distribuida, la calidad varía y no siempre existe verificación cruzada, lo que justifica pruebas sistemáticas en emuladores y hardware real

### 5. Síntesis crítica y brechas detectadas

De la revisión anterior se desprenden varias observaciones:

- Emulación madura pero heterogénea: hay emuladores robustos (PockEmul) y proyectos web accesibles, pero con enfoques y objetivos distintos (preservación, demostración, uso educativo). Falta una integración estrecha entre emulador y pipeline de desarrollo (compilación, carga, depuración paso a paso, pruebas).
- Tooling de compilación limitado: predominan BASIC y ensamblador; los intentos de alto nivel y utilidades varias no constituyen un toolchain moderno alineado con prácticas actuales (IR intermedia, optimizaciones básicas, tooling reproducible, CI). Esta carencia dificulta la experimentación didáctica y la producción de ejemplos no triviales.
- Documentación primaria disponible pero dispersa: los manuales oficiales son sólidos, pero su combinación con conocimiento comunitario requiere curación y validación experimental, en especial para detalles de temporización, periféricos (p. ej., CE‑15x) y compatibilidad de ROM.

### 6. Contribución del presente trabajo

Este TFG se posiciona en ese espacio intermedio con dos aportes complementarios:

- Un compilador desde un lenguaje de alto nivel diseñado ad hoc hacia código ejecutable para LH5801/PC‑1500, con énfasis en claridad semántica, generación determinista y soporte para bibliotecas básicas de E/S.

- Un emulador fiel y utilizable como banco de pruebas, integrado con el compilador para habilitar un bucle editar‑compilar‑ejecutar rápido, además de utilidades para carga/volcado de programas en formatos compatibles con el ecosistema existente.

Con ello, el trabajo cierra la brecha entre preservación/emulación y desarrollo activo, ofreciendo una plataforma didáctica y de investigación que facilita la exploración práctica de compiladores y arquitectura en un sistema clásico con recursos limitados.