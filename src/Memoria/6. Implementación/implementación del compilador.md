## **Implementación del compilador**

Para recrear el proceso de ejeciución de un programa BASIC en el SHARP PC-1500 es necesario estudiar el lenguaje interpretado por la máquina. Existen docenas de variaciones de BASIC debidas a las adaptaciones a hardwares diferentes y la propia evolución del lenguaje a lo largo de los años. En concreto esta calculadora usa su propia versión del lenguaje implementada en la ROM, diseñada especificamente para las limitaciones del procesador LH5801 y la arquitectura.
Pese a sus diferencias, comparte un vasto repertorio de comandos con los estandares de la época, como Microsoft-BASIC. La mayor diferencia se haya en las instrucciones de control de memoria, dada su reducida capacidad, manejo de graficos y gestión de periféricos diseñados concretamente para el modelo de SHARP.

Gracias a la información presente en el Manual Técnico es posible identificar cada una de las instrucciones de BASIC especificas y generales de la calculadora, con sus respectivas direcciones de memoria y modo de uso. Por desracias no es suficiente con eso, pues probando distintos programas y sentencias sencillas escritas en BASIC se advierten patrones extraños a la hora de interpretar el lenguaje.

-- TO DO ejemplos y explicación de lo que se traga la calculadora --

### **Tabla de instrucciones BASIC**

-- TO DO pasar tabla de instrucciones a word/latex --


### **Ejecución del compilador**

El compilador cuenta con una serie de opciones de ejecucion que añaden capas de profundidad al programa. Con ellas es posible seleccionar diferentes resultados para un mismo archivo de entrada. Gracias a esta flexibilidad resulta sencillo examinar posibles errores en cada una de las etapas del proceso de compilación por separado. De la misma manera se pueden analizar paso a paso los outputs de las fases del *lexer*, *parser* y *analizer*.

Las opciónes de ejecución del programa son las siguientes:

- **--input, -i <path>**: obligatorio para la ejecución del programa, permite definir la dirección del archivo de entrada a compilar.

- **--lex, -l** : usando de base el archivo definido como input, divide el fichero en tokens e imprime el resultado por la salida estandar.

- **--parse, -p** : usando de base el archivo definido como input, genera el árbol sintáctico en base a los tokens obtenidos por el *lexer* y lo muestra por la salida estandar. Además informa de los errores que encuentre en el proceso de *parse*.

- **--analyze, -a** :usando de base el archivo definido como input, analiza semánticamente los resultados del *parser* e imprime la tabla de símbolos por la salida estandar. De la misma forma que la opción anterior informa de los errores obtenidos en el proceso.

- **--no-header** : no obligatorio, esta opción determina si se debe incluir una cabecera, precedente al resultado de la compilación, con la longitud del programa. En caso de que no se agrege esta opción, el comportamiento por defecto incluirá la cabecera al principio del binario generado. Además el programa se encargara de informar al usuario si la cabecera ha sido creada o no, a través de la salida estandar.

- **--output, -o <path>**: no obligatorio, permite elegir una ruta de destino para el resultado final de la compilación. El valor por defecto para el nombre del archivo de salida es *"a.bin"*.

- **--preserve_source_wording, -w** : no obligatorio, en presencia de esta opción el archivo de output preservará las parentesis del codigo de origen.

- **--program_name, -n <name>**: no obligatorio, permite cambiar el nombre del programa en la cabecera de salida por el indicado en <name>. La opción por defecto es el nombre del archivo de entrada junto a su extensión.

La opción por defecto: 

Ejemplo de ejecución del parser para obtener el AST:

Ejemplo de ejecución del compilador para obtener los binarios:

Este planteamiento resulta de utilidad para el fin didáctico del trabajo. La subdivisión clara y coherente permite analizar cada fase por separado y entender los mecanismos que emplea la calculadora original para interpretar los programas en BASIC. 

### **Lexer**

El lexer es una máquina de estados finita implementada como un Iterator que procesa carácter por carácter el código fuente BASIC y produce tokens (SpannedToken). Cada token incluye tanto el tipo de token como su ubicación en el código fuente (span) para mejorar los mensajes de error.

#### **2. Tipos de Tokens que Reconoce**

El lexer reconoce 6 tipos principales de tokens:

Keywords: Palabras reservadas de BASIC (FOR, IF, PRINT, etc.)
Symbols: Operadores y símbolos (+, -, *, =, <>, etc.)
Identifiers: Variables (máximo 2 caracteres, opcionalmente con $)
DecimalNumber: Números decimales (incluyendo notación científica)
BinaryNumber: Números hexadecimales (con prefijo &)
StringLiteral: Cadenas de texto entre comillas
Remark: Comentarios (líneas que empiezan con REM)

3. Inicialización y Estado

4. Proceso de Tokenización


Ignora espacios, tabs y otros whitespace excepto \n
Los newlines son significativos en BASIC (marcan fin de línea)

´´´
while let Some(&ch) = self.peek() {
    if ch.is_whitespace() && ch != '\n' {
        self.advance();  // Saltar espacios, tabs, etc.
    } else {
        break;
    }
}
´´´

El lexer usa pattern matching sobre el primer carácter para determinar qué tipo de token procesar:
´´´
'+' => Token::Symbol(Symbol::Add),
'-' => Token::Symbol(Symbol::Sub),
'*' => Token::Symbol(Symbol::Mul),
// etc.
´´´
Operadores de comparación (pueden ser de 1 o 2 caracteres):
´´´
'<' => {
    if self.peek() == Some(&'=') {      // <=
        self.advance();
        Token::Symbol(Symbol::Leq)
    } else if self.peek() == Some(&'>') { // <>
        self.advance();
        Token::Symbol(Symbol::Neq)
    } else {                            // <
        Token::Symbol(Symbol::Lt)
    }
}
´´´

Paso 3: Procesamiento de Literales de Cadena
´´´
'"' => {
    let mut string_content = String::new();
    let mut found_closing_quote = false;
    
    while let Some(&ch) = self.peek() {
        if ch == '"' {
            self.advance();
            found_closing_quote = true;
            break;
        }
        if ch == '\n' {
            found_closing_newline = true;
            break;  // Newline termina la cadena
        }
        string_content.push(ch);
        self.advance();
    }
    
    // Crear token StringLiteral con flag de si se cerró correctamente
}
´´´
Paso 4: Procesamiento de Números Hexadecimales
´´´
'&' => {
    let mut hex_digits = String::from("&");
    while let Some(&ch) = self.peek() {
        if ch.is_ascii_hexdigit() {
            hex_digits.push(ch.to_ascii_uppercase());
            self.advance();
        } else {
            break;
        }
    }
    // Parsear como BinaryNumber
}
´´´
Paso 5: Procesamiento de Números Decimales
´´´
ch if ch.is_ascii_digit() || ch == '.' => {
    let mut number_str = String::new();
    number_str.push(ch);
    
    let mut has_dot = ch == '.';
    let mut has_e = false;
    
    while let Some(&ch) = self.peek() {
        if ch.is_ascii_digit() {
            // Agregar dígito
        } else if ch == '.' && !has_dot && !has_e {
            // Punto decimal (solo uno)
        } else if ch == 'E' && !has_e {
            // Notación científica con lookahead
            // para no confundir con keywords que empiezan con E
        }
        // etc.
    }
}
´´´
Paso 6: Procesamiento Complejo de Identificadores y Keywords
Esta es la parte más compleja del lexer. El problema es que en BASIC clásico, las palabras no necesitan estar separadas por espacios. Por ejemplo:

*FORTO* debe tokenizarse como *FOR + TO*
*IFAGOSUB3* debe tokenizarse como *IF + A + GOSUB + 3*
Algoritmo de reconocimiento:

Lectura greedy: Lee todos los caracteres alfanuméricos posibles
Búsqueda de keywords por sufijo: Busca en LONGEST_SUFFIX_FIRST si alguna keyword es sufijo de la palabra leída
Partición: Si encuentra una keyword como sufijo, divide la palabra en identificador + keyword

´´´
// Leer palabra completa
let mut word = String::new();
word.push(ch.to_ascii_uppercase());

// Lookahead complejo para leer caracteres alfanuméricos
{
    let cloned_iter = self.input.clone();
    for (i, next_ch) in cloned_iter.enumerate() {
        if (i < 1 && next_ch.is_ascii_alphanumeric())
            || next_ch.is_ascii_alphabetic()
        {
            word.push(next_ch.to_ascii_uppercase());
        } else if next_ch == '$' || next_ch == '#' {
            word.push(next_ch);
            break;
        } else {
            break;
        }
    }
}

// Buscar keywords como sufijo
for keyword in Keyword::LONGEST_SUFFIX_FIRST.iter() {
    if word.ends_with(keyword) {
        let keyword_len = keyword.len();
        let ident_len = word.len() - keyword_len;
        
        if ident_len == 0 {
            // Toda la palabra es una keyword
            return keyword_token;
        } else {
            // Palabra = identificador + keyword
            // Devolver primero el identificador
            return identifier_token;
        }
    }
}
´´´

Paso 7: Manejo Especial de Comentarios REM
Cuando se detecta la keyword REM, el lexer cambia a un modo especial:

´´´
if kw == Keyword::Rem {
    let mut comment = String::new();
    
    // El espacio después de REM es obligatorio
    if self.advance() != Some(' ') {
        return error;
    }
    
    // Según el modo de comentario configurado
    match self.remark_opt {
        RemarkLexOption::TrimWhitespace => {
            // Saltar espacios adicionales
        }
        RemarkLexOption::KeepWhole => {
            // Mantener todo el contenido
        }
    }
    
    // Leer hasta fin de línea
    while let Some(&next_ch) = self.peek() {
        if next_ch == '\n' {
            break;
        }
        comment.push(next_ch);
        self.advance();
    }
}
´´´
5. Gestión de Posiciones y Spans
El lexer rastrea la posición en bytes para crear spans precisos:
´´´
fn advance(&mut self) -> Option<char> {
    if let Some(ch) = self.input.next() {
        self.position += ByteOffset::from(ch.len_utf8() as i64);
        Some(ch)
    } else {
        None
    }
}

fn span_from(&self, start: ByteIndex) -> Span {
    Span::new(start, self.position)
}
´´´
6. Manejo de Errores
El lexer puede generar varios tipos de errores:

UnterminatedStringLiteral: Cadena sin cerrar
InvalidBinaryNumber: Número hexadecimal inválido
UnexpectedCharacter: Carácter no reconocido
InvalidRemark: Comentario REM sin espacio

7. Integración con el Compilador Principal
En main.rs, el lexer se usa así:

´´´
let lexer = Lexer::new(&input, args.remark_mode.into());
let mut tokens: Vec<SpannedToken> = Vec::new();

for token_result in lexer {  // Iterator protocol
    match token_result {
        Ok(spanned_token) => tokens.push(spanned_token),
        Err(lex_error) => {
            let compile_error = CompileError::from(lex_error);
            print_error(&compile_error, &filename, &input);
            std::process::exit(1);
        }
    }
}
´´´
8. Características Especiales de BASIC
Este lexer está diseñado específicamente para BASIC y maneja sus peculiaridades:

Keywords sin separación: FORTO → FOR + TO
Variables cortas: Máximo 2 caracteres + opcional $
Números hexadecimales: Prefijo &
Comentarios especiales: REM requiere espacio después
Newlines significativos: \n es un token importante
Símbolos especiales: √ para raíz cuadrada, π para pi

Este lexer es bastante sofisticado y maneja casos edge muy específicos del BASIC clásico, especialmente la capacidad de reconocer keywords y identificadores que no están separados por espacios, lo cual es crucial para la compatibilidad con el lenguaje BASIC histórico.

### **Parser**

1. Arquitectura General del Parser
El parser del compilador BASIC implementa un analizador sintáctico descendente recursivo con las siguientes características principales:

Estructura Principal
´´´
pub struct Parser<I: Clone>
where
    I: Iterator<Item = SpannedToken>,
{
    tokens: Peekable<I>,
}
´´´
Entrada: Stream de tokens con información de posición (SpannedToken)
Método: Descenso recursivo con lookahead de 1 token
Salida: AST (Árbol de Sintaxis Abstracta) representado como Program
Manejo de errores: Recuperación de errores con continuación del análisis

2. Jerarquía del AST y Estructuras de Datos
Programa
´´´
pub struct Program {
    lines: BTreeMap<u16, CodeLine>,
}
´´´

Organización: Líneas ordenadas por número de línea (BTreeMap)
Ventaja: Inserción automática en orden y acceso eficiente

Línea de Código
´´´
pub struct CodeLine {
    number: u16,               // Número de línea obligatorio
    label: Option<CodeLineLabel>, // Etiqueta opcional "STRING"
    statements: Vec<Statement>,   // Lista de sentencias separadas por ':'
}
´´´

Sentencias y Expresiones
´´´
pub struct Statement {
    inner: StatementInner,  // Contenido específico de la sentencia
    span: Span,            // Posición en el código fuente
}

pub struct Expr {
    inner: ExprInner,      // Contenido específico de la expresión
    span: Span,           // Posición en el código fuente
}
´´´
3. Proceso de Análisis Sintáctico
3.1 Análisis de Programa Completo
´´´
pub fn parse_with_error_recovery(&mut self) -> (Program, Vec<ParseError>)
´´´

Algoritmo principal:

Inicializar programa vacío y lista de errores
Mientras haya tokens:
- Intentar parsear una línea de código
- Si éxito: agregar línea al programa
- Si error: agregar error a la lista y saltar a próxima línea
Retornar programa y errores encontrados

Recuperación de errores:
´´´
fn skip_to_next_code_line(&mut self) {
    loop {
        match self.peek_token() {
            Token::Symbol(Symbol::Newline) => { /* saltar newlines */ }
            Token::Symbol(Symbol::Eof) => { break; }
            _ => { self.next_spanned(); /* saltar token */ }
        }
    }
}
´´´

3.2 Análisis de Línea de Código
´´´
fn parse_code_line_with_recovery(&mut self) -> ParseResult<Option<CodeLine>>
´´´

Estructura de línea BASIC:
´´´
<número> ["etiqueta"[:]] <sentencia1>[: <sentencia2>...] <newline>
´´´

Proceso:

Verificar número de línea (obligatorio)
Parsear etiqueta opcional: "STRING" con : opcional
Parsear sentencias separadas por :
Consumir newline o EOF
3.3 Análisis de Sentencias
El parser reconoce más de 50 tipos diferentes de sentencias BASIC:
´´´
fn parse_statement(&mut self, is_let_mandatory: bool) -> ParseResult<Option<Statement>>
´´´

Ejemplos de sentencias implementadas:

Control de flujo: *IF, FOR, NEXT, GOTO, GOSUB, ON*
Entrada/Salida: *PRINT, INPUT, LPRINT, GPRINT*
Variables: *LET, DIM, READ, DATA*
Gráficos: *LINE, RLINE, COLOR, CURSOR*
Sistema: *CLEAR, CLS, BEEP, WAIT*

Estrategia de parsing:

Lookahead: Inspeccionar primer token para determinar tipo
Delegación: Cada tipo de sentencia tiene su función específica
Fallback: LET se intenta al final (asignaciones implícitas)

3.4 Análisis de Expresiones con Precedencia
El parser implementa precedencia de operadores correcta mediante funciones recursivas específicas para cada nivel:
´´´
// Jerarquía de precedencia (de menor a mayor):
parse_and_or_expr()        // OR, AND
parse_unary_logical_expr() // NOT
parse_comparison_expr()    // =, <>, <, >, <=, >=
parse_add_sub_expr()       // +, -
parse_mul_div_expr()       // *, /
parse_unary_op_expr()      // unary +, -
parse_exponent_expr()      // ^
parse_expression_factor()  // números, variables, funciones, paréntesis
´´´
Algoritmo para operadores binarios (ejemplo con suma/resta):
´´´
fn parse_add_sub_expr(&mut self) -> ParseResult<Option<Expr>> {
    if let Some(mut left) = self.parse_mul_div_expr()? {
        while let Some(op) = self.next_if_token(|token| {
            matches!(token, Token::Symbol(Symbol::Add) | Token::Symbol(Symbol::Sub))
        }) {
            let right = self.expect_mul_div_expr()?;
            let binary_op = match op.token() {
                Token::Symbol(Symbol::Add) => BinaryOp::Add,
                Token::Symbol(Symbol::Sub) => BinaryOp::Sub,
                _ => unreachable!(),
            };
            left = Expr::new(
                ExprInner::Binary(Box::new(left), binary_op, Box::new(right)),
                left.span().merge(right.span()),
            );
        }
        Ok(Some(left))
    } else {
        Ok(None)
    }
}
´´´

Características importantes:

Asociatividad izquierda: *a + b + c → (a + b) + c*
Precedencia correcta: *2 + 3 * 4 → 2 + (3 * 4)*
Paréntesis: Alteran precedencia correctamente
4. Manejo de Elementos Específicos de BASIC
4.1 Sentencia *IF* Compleja
´´´
fn parse_if_stmt(&mut self) -> ParseResult<Option<Statement>>
´´´

Formas soportadas:
´´´
IF condition THEN statement
IF condition statement          // THEN implícito
IF condition THEN line_number   // GOTO implícito
´´´

Lógica de parsing:

Parsear condición (expresión)
Verificar *THEN* opcional
Intentar parsear sentencia completa
Si falla, parsear expresión (número de línea para *GOTO*)

4.2 Sentencias *FOR/NEXT*
´´´
fn parse_for_stmt(&mut self) -> ParseResult<Option<Statement>>

´´´
Sintaxis:
´´´
FOR variable = start TO end [STEP increment]
´´´

Proceso:

Parsear asignación inicial: *variable = start*
Esperar keyword *TO*
Parsear expresión final
Parsear *STEP* opcional con su expresión

4.3 Funciones Built-in
´´´
fn parse_function(&mut self) -> ParseResult<Option<Function>>
´´´

Tipos de funciones:

Unarias: *ABS(x), INT(x), SIN(x), COS(x), etc*.
Con parámetros: *MID$(string, start, length)*
Sistema: *PEEK(address), RND(range)*

Ejemplo *MID$*:
´´´
Token::Keyword(Keyword::MidDollar) => {
    self.tokens.next();
    self.expect_token(&Token::Symbol(Symbol::LParen), "Expected '(' after MID$")?;
    let string = self.expect_expression()?;
    self.expect_token(&Token::Symbol(Symbol::Comma), "Expected ',' after MID$ string")?;
    let start_pos = self.expect_expression()?;
    self.expect_token(&Token::Symbol(Symbol::Comma), "Expected ',' after MID$ start")?;
    let length = self.expect_expression()?;
    self.expect_token(&Token::Symbol(Symbol::RParen), "Expected ')' after MID$ length")?;
    // ... crear Function
}
´´´
4.4 L-Values (Variables y Referencias)
´´´
fn parse_lvalue(&mut self) -> ParseResult<Option<LValue>>
´´´

Tipos soportados:

Variables simples: A, B$, X1
Arrays 1D: A(5), B$(I)
Arrays 2D: M(I,J)
Memoria fija: @(address), @$(address)
Built-ins: TIME, INKEY$, PI
5. Métodos Utilitarios del Parser

5.1 Gestión de Tokens
´´´
fn peek_token(&mut self) -> &Token                    // Ver próximo token
fn next_spanned(&mut self) -> Option<SpannedToken>   // Consumir token
fn next_if_token_eq(&mut self, expected: &Token) -> Option<SpannedToken> // Consumir si coincide
fn expect_token(&mut self, expected: &Token, context: &str) -> ParseResult<SpannedToken> // Elegir Token
´´´

5.2 Gestión de Errores y Spans
´´´
fn current_span(&mut self) -> Span                   // Posición actual
´´´
Cada nodo del AST incluye:

Span completo: Desde inicio hasta final del construct
Merge de spans: Combinación de posiciones de sub-elementos
´´´
let span = start_span.merge(end_span);
´´´

6. Generación de Código
6.1 Serialización a Bytecode
´´´
fn write_bytes(&self, bytes: &mut Vec<u8>, preserve_source_wording: bool)
´´´

Formato del programa:
´´´
[línea1][línea2]...[línean][0xFF]
´´´

Formato de cada línea:
´´´
[número_línea:2bytes][tamaño:1byte]["etiqueta"][sentencias][0x0D]
´´´

6.2 Regeneración de Código Fuente
´´´
fn show(&self, preserve_source_wording: bool) -> String
´´´

Características:

preserve_source_wording: Mantener formato original vs. normalizado
Precedencia: Inserción automática de paréntesis cuando necesario
Formato: Espaciado y estructura coherente

7. Manejo de Errores Avanzado
7.1 Tipos de Error
´´´
pub enum ParseError {
    UnexpectedToken { expected: String, found: String, span: Span },
    UnexpectedEndOfInput { expected: String, span: Span },
    ExpectedExpression { span: Span },
    ExpectedLValue { span: Span },
    MissingToken { token: String, span: Span },
    InvalidExpression { message: String, span: Span },
    // ... más tipos
}
´´´

7.2 Recuperación de Errores
Filosofía: Continuar parsing después de errores para encontrar múltiples problemas

Estrategias:

Error en sentencia: Saltar al próximo : o newline
Error en línea: Saltar a próxima línea numerada
Error en expresión: Intentar recuperar en próximo operador


8. Testing y Validación
8.1 Tests de Precedencia
El parser incluye tests extensivos para validar:

Precedencia correcta entre operadores
Asociatividad izquierda para operadores del mismo nivel
Efecto de paréntesis en precedencia
Ejemplo de test:
´´´
#[test]
fn test_operator_precedence_chain() {
    // 1 + 2 * 3 ^ 4 debe parsear como 1 + (2 * (3 ^ 4))
    let expr = parse_expression_from_str("1 + 2 * 3 ^ 4").unwrap();
    // Verificaciones detalladas de estructura del AST...
}
´´´
9. Integración con el Compilador
9.1 Pipeline de Compilación
´´´
// En main.rs:
let mut parser = Parser::new(tokens.into_iter());
let (program, parse_errors) = parser.parse_with_error_recovery();

// Reportar errores pero continuar si hay líneas válidas
for parse_error in &parse_errors {
    let compile_error = CompileError::from(parse_error.clone());
    print_error(&compile_error, &filename, &input);
}
´´´

9.2 Modos de Operación
--parse: Solo mostrar AST parseado
--analyze: Continuar con análisis semántico
Compilación completa: Generar bytecode final

10. Fortalezas y Características Únicas

10.1 Recuperación de Errores Robusta
Continúa parsing después de errores
Reporta múltiples problemas en una pasada
Preserva líneas válidas para continuar compilación

10.2 Preservación del Formato Original
Opción preserve_source_wording mantiene formato exacto
Útil para herramientas de refactoring y formateo

10.3 Compatibilidad BASIC Histórica
Soporte para todas las construcciones BASIC clásicas
Manejo correcto de sintaxis ambigua (IF con/sin THEN)
Números de línea obligatorios (como BASIC original)

10.4 Información de Posición Detallada
Cada nodo incluye span completo
Mensajes de error precisos
Soporte para herramientas de desarrollo (IDE, debugger)

El parser implementa un analizador sintáctico profesional y robusto que maneja correctamente la complejidad del lenguaje BASIC clásico. Su arquitectura de descenso recursivo con recuperación de errores, combinada con el manejo preciso de precedencia y las características específicas de BASIC, lo convierte en una base sólida para un compilador completo.

La implementación demuestra un profundo entendimiento tanto de las técnicas de parsing modernas como de las peculiaridades históricas del BASIC, resultando en un parser que es tanto técnicamente correcto como prácticamente útil.

### **Analizador sintáctico**

El análisis semántico es la tercera fase del compilador BASIC (después del análisis léxico y sintáctico), responsable de verificar la **corrección semántica** del programa y construir la **tabla de símbolos**. Este analizador implementa un sistema de tipos estático que valida:

- **Concordancia de tipos** en asignaciones y expresiones
- **Declaración y uso correcto** de variables y arrays
- **Validación de argumentos** de funciones built-in
- **Construcción de tabla de símbolos** para análisis posteriores

**Arquitectura Principal**

```rust
pub struct SemanticAnalyzer {
    symbol_table: SymbolTable,
}

pub struct SymbolTable {
    variables: HashMap<Identifier, BasicType>,  // Variables y arrays
    labels: HashMap<String, u16>,               // Etiquetas de línea
}
```

2. Sistema de Tipos
2.1 Tipos Básicos

El analizador define un sistema de tipos jerárquico:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum BasicType {
    Numeric,           // Variables numéricas (A, B1, X)
    String,            // Variables de cadena (A$, B1$, X$)
    NumericArray,      // Arrays numéricos 1D (A(), B())
    StringArray,       // Arrays de cadenas 1D (A$(), B$())
    Numeric2DArray,    // Arrays numéricos 2D (A(,), B(,))
    String2DArray,     // Arrays de cadenas 2D (A$(,), B$(,))
}
```

2.2 Inferencia de Tipos

**Variables y L-Values:**
- **Sufijo `$`**: Determina tipo String
- **Sin sufijo**: Determina tipo Numeric
- **Acceso a array**: Infiere tipo del elemento basado en el identificador

```rust
fn analyze_lvalue(&mut self, lvalue: &LValue) -> SemanticResult<BasicType> {
    match &lvalue.inner {
        LValueInner::Identifier(identifier) => {
            let var_type = if identifier.has_dollar() {
                BasicType::String    // Variables con $: A$, B1$
            } else {
                BasicType::Numeric   // Variables sin $: A, B1, X
            };
            self.symbol_table.insert_variable(*identifier, var_type);
            Ok(var_type)
        }
        // ... otros casos
    }
}
```

**Arrays:**
```rust
LValueInner::Array1DAccess { identifier, index } => {
    // Verificar que el índice sea numérico
    let index_type = self.analyze_expression(index)?;
    if index_type != BasicType::Numeric {
        return Err(SemanticError::InvalidArrayIndex { ... });
    }
    
    // Determinar tipo del array y elemento
    let array_type = if identifier.has_dollar() {
        BasicType::StringArray
    } else {
        BasicType::NumericArray
    };
    
    let element_type = if identifier.has_dollar() {
        BasicType::String
    } else {
        BasicType::Numeric
    };
    
    Ok(element_type)
}
```

3. Proceso de Análisis
3.1 Análisis de Programa Completo

```rust
pub fn analyze_program(&mut self, program: &Program) -> SemanticResult<()> {
    // PRIMERA PASADA: Recolectar etiquetas
    for line in program.lines() {
        if let Some(label) = line.label() {
            self.symbol_table.insert_label(label.to_owned(), line.number());
        }
    }

    // SEGUNDA PASADA: Analizar sentencias
    for line in program.lines() {
        self.analyze_line(line)?;
    }

    Ok(())
}
```

**Estrategia de dos pasadas:**
1. **Pasada 1**: Recolectar todas las etiquetas de línea para permitir saltos hacia adelante
2. **Pasada 2**: Análisis semántico completo de todas las sentencias

3.2 Análisis de Sentencias

El analizador maneja **más de 25 tipos diferentes** de sentencias BASIC:

```rust
fn analyze_statement(&mut self, statement: &Statement) -> SemanticResult<()> {
    match &statement.inner {
        StatementInner::Let { inner, .. } => { ... }
        StatementInner::If { condition, then_stmt, .. } => { ... }
        StatementInner::For { assignment, to_expr, step_expr } => { ... }
        // ... casos específicos para cada tipo de sentencia
    }
}
```

Ejemplos Detallados:

**Sentencia LET (Asignación):**
```rust
StatementInner::Let { inner, .. } => {
    for assignment in inner.assignments() {
        self.analyze_assignment(assignment)?;
    }
}

fn analyze_assignment(&mut self, assignment: &Assignment) -> SemanticResult<()> {
    let lvalue_type = self.analyze_lvalue(assignment.lvalue())?;
    let expr_type = self.analyze_expression(assignment.expr())?;

    // Verificar concordancia de tipos
    if lvalue_type != expr_type {
        return Err(SemanticError::TypeMismatch {
            expected: lvalue_type,
            found: expr_type,
            span: assignment.expr().span(),
        });
    }

    Ok(())
}
```

**Sentencia FOR:**
```rust
StatementInner::For { assignment, to_expr, step_expr } => {
    // La asignación inicial debe ser válida
    self.analyze_assignment(assignment)?;
    
    // La expresión TO debe ser numérica
    let to_type = self.analyze_expression(to_expr)?;
    if to_type != BasicType::Numeric {
        return Err(SemanticError::TypeMismatch {
            expected: BasicType::Numeric,
            found: to_type,
            span: to_expr.span(),
        });
    }
    
    // STEP opcional debe ser numérico
    if let Some(step) = step_expr {
        let step_type = self.analyze_expression(step)?;
        if step_type != BasicType::Numeric {
            return Err(SemanticError::TypeMismatch { ... });
        }
    }
}
```

**Sentencia IF:**
```rust
StatementInner::If { condition, then_stmt, .. } => {
    // NOTA: En BASIC, la condición puede ser string o numeric
    let _condition_type = self.analyze_expression(condition)?;
    self.analyze_statement(then_stmt)?;
}
```

3.3 Análisis de Expresiones

Jerarquía de Análisis:

```rust
fn analyze_expression(&mut self, expr: &Expr) -> SemanticResult<BasicType>
fn analyze_expr_inner(&mut self, expr_inner: &ExprInner) -> SemanticResult<BasicType>
```

Tipos de Expresiones:

**Literales:**
```rust
ExprInner::DecimalNumber(_) => Ok(BasicType::Numeric),
ExprInner::BinaryNumber(_) => Ok(BasicType::Numeric),
ExprInner::StringLiteral { .. } => Ok(BasicType::String),
```

**Operadores Unarios:**
```rust
ExprInner::Unary(op, expr) => {
    let expr_type = self.analyze_expression(expr)?;
    match op {
        UnaryOp::Plus | UnaryOp::Minus => {
            if expr_type != BasicType::Numeric {
                return Err(SemanticError::TypeMismatch { ... });
            }
            Ok(BasicType::Numeric)
        }
        UnaryOp::Not => {
            if expr_type != BasicType::Numeric {
                return Err(SemanticError::TypeMismatch { ... });
            }
            Ok(BasicType::Numeric)
        }
    }
}
```

**Operadores Binarios:**
```rust
ExprInner::Binary(left, op, right) => {
    let left_type = self.analyze_expression(left)?;
    let right_type = self.analyze_expression(right)?;

    match op {
        // Suma: permite numeric + numeric O string + string
        BinaryOp::Add => {
            if left_type != right_type {
                return Err(SemanticError::TypeMismatch { ... });
            }
            Ok(left_type)  // Retorna el tipo común
        }
        
        // Operaciones aritméticas: solo numeric
        BinaryOp::Sub | BinaryOp::Mul | BinaryOp::Div | BinaryOp::Exp => {
            if left_type != BasicType::Numeric || right_type != BasicType::Numeric {
                return Err(SemanticError::TypeMismatch { ... });
            }
            Ok(BasicType::Numeric)
        }
        
        // Comparaciones: tipos matching, retorna numeric (booleano)
        BinaryOp::Eq | BinaryOp::Neq | BinaryOp::Lt | 
        BinaryOp::Gt | BinaryOp::Leq | BinaryOp::Geq => {
            if left_type != right_type {
                return Err(SemanticError::TypeMismatch { ... });
            }
            Ok(BasicType::Numeric)  // Booleanos representados como numeric
        }
        
        // Operadores lógicos: solo numeric
        BinaryOp::And | BinaryOp::Or => {
            if left_type != BasicType::Numeric || right_type != BasicType::Numeric {
                return Err(SemanticError::TypeMismatch { ... });
            }
            Ok(BasicType::Numeric)
        }
    }
}
```

3.4 Análisis de Funciones Built-in

El analizador valida **más de 20 funciones built-in**:

```rust
fn analyze_function_call(&mut self, function: &Function) -> SemanticResult<BasicType> {
    match &function.inner {
        // Funciones de cadenas
        FunctionInner::Mid { string, start, length } => {
            let string_type = self.analyze_expression(string)?;
            let start_type = self.analyze_expression(start)?;
            let length_type = self.analyze_expression(length)?;

            if string_type != BasicType::String {
                return Err(SemanticError::InvalidFunctionArgument {
                    span: string.span(),
                    function_name: "MID$".to_string(),
                    message: "first argument must be string".to_string(),
                });
            }
            // Verificar argumentos numéricos...
            Ok(BasicType::String)
        }
        
        // Funciones matemáticas
        FunctionInner::Sin { expr } => {
            let expr_type = self.analyze_expression(expr)?;
            if expr_type != BasicType::Numeric {
                return Err(SemanticError::InvalidFunctionArgument { ... });
            }
            Ok(BasicType::Numeric)
        }
        
        // Funciones de conversión
        FunctionInner::Val { expr } => {  // String → Numeric
            let expr_type = self.analyze_expression(expr)?;
            if expr_type != BasicType::String {
                return Err(SemanticError::InvalidFunctionArgument { ... });
            }
            Ok(BasicType::Numeric)
        }
        
        FunctionInner::Str { expr } => {  // Numeric → String
            let expr_type = self.analyze_expression(expr)?;
            if expr_type != BasicType::Numeric {
                return Err(SemanticError::InvalidFunctionArgument { ... });
            }
            Ok(BasicType::String)
        }
    }
}
```

Clasificación de Funciones:

**Funciones Matemáticas** (Numeric → Numeric):
- `ABS(x)`, `INT(x)`, `SGN(x)`
- `SIN(x)`, `COS(x)`, `TAN(x)`
- `LN(x)`, `LOG(x)`, `SQR(x)`
- `RND(x)`, `POINT(x)`

**Funciones de Cadenas** (String → ...):
- `LEN(s$)` → Numeric
- `ASC(s$)` → Numeric
- `VAL(s$)` → Numeric

**Funciones de Conversión**:
- `STR$(n)` → String
- `CHR$(n)` → String

**Funciones de Manipulación de Cadenas**:
- `MID$(s$, start, length)` → String
- `LEFT$(s$, length)` → String
- `RIGHT$(s$, length)` → String

3.5 Análisis de Declaraciones DIM

```rust
fn analyze_dim_declaration(&mut self, dim: &DimInner) -> SemanticResult<()> {
    match dim {
        DimInner::DimInner1D { identifier, size, string_length, .. } => {
            // Verificar que el tamaño sea numérico
            let size_type = self.analyze_expression(size)?;
            if size_type != BasicType::Numeric {
                return Err(SemanticError::TypeMismatch { ... });
            }

            // Verificar longitud de cadena opcional
            if let Some(str_len) = string_length {
                let str_len_type = self.analyze_expression(str_len)?;
                if str_len_type != BasicType::Numeric {
                    return Err(SemanticError::TypeMismatch { ... });
                }
            }

            // Registrar el array en la tabla de símbolos
            let array_type = if identifier.has_dollar() {
                BasicType::StringArray
            } else {
                BasicType::NumericArray
            };
            self.symbol_table.insert_variable(*identifier, array_type);
        }
        
        DimInner::DimInner2D { identifier, rows, cols, string_length, .. } => {
            // Similar pero para arrays 2D...
        }
    }
    Ok(())
}
```

4. Gestión de Errores Semánticos
4.1 Tipos de Error

```rust
pub enum SemanticError {
    TypeMismatch {
        expected: BasicType,
        found: BasicType,
        span: Span,
    },
    InvalidArrayIndex {
        message: String,
        span: Span,
    },
    InvalidFunctionArgument {
        function_name: String,
        message: String,
        span: Span,
    },
    InvalidExpression {
        message: String,
        span: Span,
    },
}
```

4.2 Ejemplos de Detección de Errores

**Error de Tipo:**
```basic
10 A$ = 42          ' ERROR: String variable con valor numeric
20 B = "HELLO"      ' ERROR: Numeric variable con valor string
30 C = A + "WORLD"  ' ERROR: Tipos incompatibles en suma
```

**Error de Índice:**
```basic
10 DIM A(10)
20 B = A("X")       ' ERROR: Índice debe ser numeric, no string
```

**Error de Función:**
```basic
10 X = SIN("45")    ' ERROR: SIN requiere argumento numeric
20 Y$ = LEN(42)     ' ERROR: LEN requiere argumento string
```

4.3 Reporting de Errores
Los errores incluyen **información de posición precisa**:

```rust
return Err(SemanticError::TypeMismatch {
    expected: BasicType::Numeric,
    found: found_type,
    span: expr.span(),  // Posición exacta en el código fuente
});
```
5. Tabla de Símbolos

5.1 Estructura

```rust
#[derive(Debug, Clone)]
pub struct SymbolTable {
    variables: HashMap<Identifier, BasicType>,
    labels: HashMap<String, u16>,
}
```

5.2 Funciones de la Tabla

**Registro de Variables:**
```rust
pub fn insert_variable(&mut self, id: Identifier, type_: BasicType) {
    self.variables.insert(id, type_);
}
```

**Registro de Etiquetas:**
```rust
pub fn insert_label(&mut self, name: String, line_number: u16) {
    self.labels.insert(name, line_number);
}
```

5.3 Información Recolectada

**Variables:**
- **Identificador**: A, B1, X$, etc.
- **Tipo**: Numeric, String, NumericArray, etc.
- **Declaración implícita**: Variables se declaran en primer uso

**Etiquetas:**
- **Nombre**: String literal de la etiqueta
- **Línea**: Número de línea donde aparece

**Ejemplo de tabla resultante:**
```
Variables:
  A     → Numeric
  B$    → String
  C     → NumericArray
  D$    → StringArray
  
Labels:
  "LOOP"    → 100
  "ERROR"   → 9000
```

6. Integración con el Pipeline de Compilación

6.1 Invocación desde main.rs

```rust
if args.analyze {
    let mut parser = Parser::new(tokens.into_iter());
    let (program, parse_errors) = parser.parse_with_error_recovery();

    // Reportar errores de parsing...
    
    match analyze_program(&program) {
        Ok(symbol_table) => {
            println!("Semantic analysis completed successfully!");
            println!("Symbol table: {symbol_table:#?}");
        }
        Err(semantic_error) => {
            let compile_error = CompileError::from(semantic_error);
            print_error(&compile_error, &filename, &input);
            std::process::exit(1);
        }
    }
}
```

6.2 Función Pública de Análisis

```rust
/// Función pública para analizar un programa completo
pub fn analyze_program(program: &Program) -> SemanticResult<SymbolTable> {
    let mut analyzer = SemanticAnalyzer::new();
    analyzer.analyze_program(program)?;
    Ok(analyzer.symbol_table)
}
```

7. Características Especiales del BASIC
7.1 Peculiaridades del Lenguaje

**Variables Implícitas:**
- No requieren declaración explícita
- Tipo inferido por sufijo `$`
- Registradas automáticamente en primer uso

**Flexibilidad de IF:**
- Condiciones pueden ser string o numeric
- String vacío = false, no vacío = true
- Numeric 0 = false, no-cero = true

**Arrays:**
- `DIM` opcional para arrays pequeños
- Índices siempre numéricos
- Longitud de cadena opcional: `DIM A$(10)*20`

7.2 Compatibilidad Histórica

**Operador `+` Polimórfico:**
```basic
10 A = 5 + 3        ' Suma numérica → 8
20 B$ = "HO" + "LA" ' Concatenación → "HOLA"
30 C = A + "X"      ' ERROR: Tipos incompatibles
```

**Variables Built-in:**
- `TIME`: Variable numérica del sistema
- `INKEY$`: Variable string del sistema
- `PI`: Constante numérica

8. Testing y Validación

8.1 Test Suite

```rust
#[test]
fn test_numeric_assignment() {
    let mut analyzer = SemanticAnalyzer::new();
    let identifier = Identifier::new('X', None, false).unwrap();
    
    let assignment = Assignment::new(
        LValue::new(LValueInner::Identifier(identifier), placeholder_span()),
        Box::new(Expr::new(
            ExprInner::DecimalNumber(DecimalNumber::new(42.0)),
            placeholder_span(),
        )),
        placeholder_span(),
    );

    let result = analyzer.analyze_assignment(&assignment);
    assert!(result.is_ok());
}

#[test]
fn test_type_mismatch() {
    // Test asignación string = numeric (debe fallar)
    let result = analyzer.analyze_assignment(&invalid_assignment);
    assert!(result.is_err());
    
    if let Err(SemanticError::TypeMismatch { expected, found, .. }) = result {
        assert_eq!(expected, BasicType::String);
        assert_eq!(found, BasicType::Numeric);
    }
}
```

8.2 Cobertura de Testing

- **Asignaciones válidas e inválidas**
- **Operadores con tipos correctos e incorrectos**
- **Funciones con argumentos válidos e inválidos**
- **Arrays con índices correctos e incorrectos**
- **Declaraciones DIM válidas e inválidas**

9. Extensibilidad y Mantenimiento

9.1 Agregar Nuevos Tipos

```rust
// Futuras extensiones posibles:
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum BasicType {
    // ... tipos existentes
    Complex,        // Números complejos
    Record(String), // Tipos registros definidos por usuario
    Pointer(Box<BasicType>), // Punteros
}
```

9.2 Agregar Nuevas Funciones
```rust
// En analyze_function_call:
FunctionInner::NewFunction { arg1, arg2 } => {
    let arg1_type = self.analyze_expression(arg1)?;
    let arg2_type = self.analyze_expression(arg2)?;
    // ... validación específica
    Ok(return_type)
}
```

El análisis semántico constituye una fase crítica del compilador, proporcionando una base sólida para la validación de programas BASIC y estableciendo las estructuras de datos necesarias para fases posteriores del compilador.

### **Generación de código intermedio y binarios**

La generación de bytecode es la **fase final** del compilador BASIC, responsable de convertir el AST (Árbol de Sintaxis Abstracta) validado en **código binario ejecutable** para la calculadora Sharp PC-1500. Este proceso implementa una **serialización estructurada** que preserva tanto la funcionalidad como, opcionalmente, el formato original del código fuente.

1. Diseño

El generador de bytecode sigue el principio de compilación a código fuente tokenizado, donde:
- Las palabras clave se convierten en códigos internos de 16 bits
- Los identificadores se mantienen como *texto ASCII*
- Los números se optimizan en representación textual
- Las cadenas se preservan con información de formato original

```rust
pub struct Program {
    lines: BTreeMap<u16, CodeLine>,  // Líneas ordenadas por número
}

pub struct CodeLine {
    number: u16,                     // Número de línea (2 bytes)
    label: Option<CodeLineLabel>,    // Etiqueta opcional
    statements: Vec<Statement>,      // Sentencias de la línea
}
```
2 Formato del archivo ejecutable

El archivo final tiene la siguiente estructura:

```
[HEADER: 27 bytes][PROGRAM BODY: variable][EOF MARKER: 1 byte]
```
El PC-1500 siempre espera un header con el nombre y la longitu del programa a ejecutar. Este siempre termina con la secuencia hexadecimal *0xFF*.

```rust
pub struct Header {
    magic_number: u8,        // 0x01 - Identificador de formato
    file_type: [u8; 4],     // "@COM" - Tipo de archivo BASIC
    program_name: [u8; 16], // Nombre del programa (ASCII, padding 0x00)
    start_address: [u8; 2], // 0x00c5 - Dirección de inicio estándar
    capacity: [u8; 2],      // (length-1) - Tamaño del programa menos 1
    entry_address: [u8; 2], // 0x044f - Punto de entrada estándar
}
```
Además se define la función de tranformación a bytes de la siguiente manera:
```rust
    pub fn to_bytes(&self) -> Vec<u8> {
        let mut header_bytes = vec![self.magic_number];
        header_bytes.extend_from_slice(&self.file_type);
        header_bytes.extend_from_slice(&self.program_name);
        header_bytes.extend_from_slice(&self.start_address);
        header_bytes.extend_from_slice(&self.capacity);
        header_bytes.extend_from_slice(&self.entry_address);
        header_bytes
    }
```

3. Formato del Cuerpo del Programa

Como se detalla en la documentación referente al ensamblador *LHTools* de *PockEmul* cada línea BASIC termina con el código hexadecimal *0x0D* se serializa con el siguiente formato:

```
[LINE_NUMBER: 2 bytes][LINE_SIZE: 1 byte][LABEL][STATEMENTS][0x0D]
```

Ejemplo tomado de la documentación del software LHTools (página 39):

Desde el fichero origen:

```basic
10 CLS:WAIT 0
20 FOR I=0 TO 10:PRINT I
30 NEXT I
40 BEEP 1:END
```
Tras la compilación obtendremos:

```
1
    .ORIGIN: 40C5
1   40C5 00 0A 07 F0 88                 10 CLS:WAIT 0
    3A F1 B3 30 0D
2   40CF 00 14 0E F1 A5                 20 FOR I=0 TO 10:PRINT I
    49 3D 30 F1 B1
    31 30 3A F0 97
    49 0D
3   40E0 00 1E 04 F1 9A                 30 NEXT I
    49 0D
4   40E7 00 28 07 F1 82                 40 BEEP 1:END
    31 3A F1 8E 0D
    40F1 FF                             [END BASIC MARKER]
```
Nótese que al final de cada línea de programa tenemos el código *0x0D* y al final del programa el respectivo código *0xFF*. 

Este algoritmo está implementado dentro de la función *write_bytes*:

```rust
pub fn write_bytes(&self, bytes: &mut Vec<u8>, preserve_source_wording: bool) {
    // Número de línea en big-endian
    bytes.extend_from_slice(self.number.to_be_bytes().as_slice());
    
    // Placeholder para tamaño de línea
    let line_size_pos = bytes.len();
    bytes.push(0);

    // Etiqueta opcional
    if let Some(label) = &self.label {
        bytes.push(b'"');
        bytes.extend_from_slice(label.name.as_bytes());
        if label.are_quotes_closed_in_source {
            bytes.push(b'"');
        }
        if label.has_colon_in_source {
            bytes.push(b':');
        }
    }
    
    // Sentencias separadas por ':'
    for (i, statement) in self.statements.iter().enumerate() {
        statement.write_bytes(bytes, preserve_source_wording);
        if i < self.statements.len() - 1 {
            bytes.push(b':');
        }
    }
    
    // Actualizar tamaño de línea
    let line_size = (bytes.len() - line_size_pos) as u8;
    bytes[line_size_pos] = line_size;
}
```

4. Codificación de Elementos Sintácticos

4.1 Keywords

Las keywords se convierten en códigos internos de 16 bits en formato *big-endian*:

```rust
impl Keyword {
    pub const fn internal_code(&self) -> u16 {
        match self {
            Keyword::Print => 0xF097,
            Keyword::For => 0xF1A5,
            Keyword::If => 0xF196,
            Keyword::Then => 0xF1AE,
            Keyword::Goto => 0xF192,
            Keyword::Let => 0xF198,
            // ... más de 120 keywords mapeadas
        }
    }
}

// Uso en generación:
bytes.extend_from_slice(Keyword::Print.internal_code().to_be_bytes().as_slice());
```

Ejemplos de códigos:
- `PRINT` → `0xF0 0x97`
- `FOR` → `0xF1 0xA5`
- `IF` → `0xF1 0x96`
- `THEN` → `0xF1 0xAE`

4.2 Números Decimales

Los números decimales se optimizan entre formato decimal y notación científica, tal y como se explica en el manual técnico:

```rust
impl DecimalNumber {
    pub fn write_bytes(&self, bytes: &mut Vec<u8>) {
        if self.double == 0.0 {
            bytes.push(b'0');
            return;
        }

        let mut double_str = self.double.to_string();

        // Optimización para números < 1.0: quitar '0' inicial
        if self.double.abs() < 1.0 {
            let trimmed = double_str.trim_start_matches('0');
            double_str = trimmed.to_string();
        }

        // Comparar longitud: formato normal vs científico
        let scientific_str = format!("{:E}", self.double);

        if double_str.len() > scientific_str.len() {
            bytes.extend_from_slice(scientific_str.as_bytes());
        } else {
            bytes.extend_from_slice(double_str.as_bytes());
        }
    }
}
```

Ejemplos de optimización:
- `42.0` → `"42"`
- `0.5` → `".5"`
- `1000000.0` → `"1E6"`
- `0.000001` → `"1E-6"`

4.3 Números Binarios (Hexadecimales)

Los números hexadecimales se representan en formato ASCII en mayúsculas:

```rust
impl BinaryNumber {
    pub fn write_bytes(&self, bytes: &mut Vec<u8>) {
        bytes.extend_from_slice(format!("&{:X}", self.value).as_bytes());
    }
}
```

Ejemplos:
- `&FF` → `"&FF"`
- `&1A2B` → `"&1A2B"`

4.4 Identificadores

Los identificadores se escriben directamente como *ASCII*:

```rust
impl Identifier {
    pub fn write_bytes(&self, bytes: &mut Vec<u8>) {
        bytes.push(self.char1.get());
        if let Some(char2) = self.char2 {
            bytes.push(char2.get());
        }
        if self.has_dollar {
            bytes.push(b'$');
        }
    }
}
```

Ejemplos:
- Variable `A` → `"A"`
- Variable `B1` → `"B1"`
- Variable string `X$` → `"X$"`

4.5 Literales de Cadena

Las cadenas preservan información sobre el formato original:

```rust
ExprInner::StringLiteral { value, is_quote_closed_in_source } => {
    bytes.extend_from_slice(format!("\"{value}").as_bytes());
    if *is_quote_closed_in_source {
        bytes.push(b'"');
    }
}
```

Comportamiento según `preserve_source_wording`:
- Si `preserve_source_wording = true`: Mantener formato original
- Si `preserve_source_wording = false`: Normalizar formato

Esta opción se mantiene por la necesidad inicial de replicar los binarios originales encontrados en www.pc1500.com. La gramática del lenguaje BASIC específico de la calculadora fue replicado a base de prueba y error. Los binarios representaban la base a replicar para poder entender la sintáxis del lenguaje en las etapas iniciales del proyecto. Era necesario mantener todas las paréntesis del código original para replicar los binarios. Después, en el proceso de optimización, se procesió a eliminar las paréntesis innecesarias.

5. Generación de Sentencias Específicas

5.1 Sentencia LET

La sentencia LET maneja asignaciones de variables, con lógica especial para la keyword opcional:

```rust
StatementInner::Let { inner, is_let_kw_present_in_source } => {
    // LET keyword opcional en fuente, pero contextual en bytecode
    if preserve_source_wording && *is_let_kw_present_in_source {
        bytes.extend_from_slice(Keyword::Let.internal_code().to_be_bytes().as_slice());
    }
    inner.write_bytes(bytes, preserve_source_wording);
}
```
Ejemplo:
`LET A = 5`

Desglose hexadecimal:
- `F1 98` = LET keyword (0xF198)
- `41` = 'A'
- `3D` = '='
- `35` = '5'

5.2 Sentencia IF Compleja

La sentencia IF requiere manejo especial de keywords opcionales y casos de THEN/GOTO:

```rust
StatementInner::If { condition, then_stmt, is_then_kw_present_in_source, is_goto_kw_present_in_source } => {
    bytes.extend_from_slice(Keyword::If.internal_code().to_be_bytes().as_slice());
    condition.write_bytes(bytes, preserve_source_wording);
    
    if preserve_source_wording && *is_then_kw_present_in_source {
        bytes.extend_from_slice(Keyword::Then.internal_code().to_be_bytes().as_slice());
    }
    
    match &then_stmt.inner {
        StatementInner::Let { inner, .. } => {
            // LET obligatorio en contexto IF THEN
            bytes.extend_from_slice(Keyword::Let.internal_code().to_be_bytes().as_slice());
            inner.write_bytes(bytes, preserve_source_wording);
        }
        StatementInner::Goto { target } => {
            if preserve_source_wording {
                if *is_goto_kw_present_in_source {
                    bytes.extend_from_slice(Keyword::Goto.internal_code().to_be_bytes().as_slice());
                }
            } else {
                bytes.extend_from_slice(Keyword::Then.internal_code().to_be_bytes().as_slice());
            }
            target.write_bytes(bytes, preserve_source_wording);
        }
        _ => {
            then_stmt.write_bytes(bytes, preserve_source_wording);
        }
    }
}
```

Ejemplo:
`IF X > 0 THEN A = 1`

Desglose hexadecimal:
- `F1 96` = IF keyword (0xF196)
- `58` = 'X'
- `3E` = '>'
- `30` = '0'
- `F1 AE` = THEN keyword (0xF1AE)
- `F1 98` = LET keyword (0xF198, obligatorio en contexto IF THEN)
- `41` = 'A'
- `3D` = '='
- `31` = '1'

5.3 Sentencias FOR/NEXT

El manejo de bucles FOR/NEXT incluye keywords obligatorias y expresiones STEP opcionales:

```rust
StatementInner::For { assignment, to_expr, step_expr } => {
    bytes.extend_from_slice(Keyword::For.internal_code().to_be_bytes().as_slice());
    assignment.write_bytes(bytes, preserve_source_wording);
    bytes.extend_from_slice(Keyword::To.internal_code().to_be_bytes().as_slice());
    to_expr.write_bytes(bytes, preserve_source_wording);
    
    if let Some(step) = step_expr {
        bytes.extend_from_slice(Keyword::Step.internal_code().to_be_bytes().as_slice());
        step.write_bytes(bytes, preserve_source_wording);
    }
}

StatementInner::Next { lvalue } => {
    bytes.extend_from_slice(Keyword::Next.internal_code().to_be_bytes().as_slice());
    lvalue.write_bytes(bytes, preserve_source_wording);
}
```

Ejemplo 1:

`FOR I = 1 TO 10`

Desglose hexadecimal:
- `F1 A5` = FOR keyword (0xF1A5)
- `49` = 'I'
- `3D` = '='
- `31` = '1'
- `F1 B1` = TO keyword (0xF1B1)
- `31 30` = "10"

Ejemplo 2:

`FOR J = 0 TO 100 STEP 5`

Desglose hexadecimal :
- `F1 A5` = FOR keyword (0xF1A5)
- `4A` = 'J'
- `3D` = '='
- `30` = '0'
- `F1 B1` = TO keyword (0xF1B1)
- `31 30 30` = "100"
- `F1 A9` = STEP keyword (0xF1A9)
- `35` = '5'

5.4 Sentencias DIM (Declaración de Arrays)

DIM maneja declaraciones de arrays 1D y 2D con longitudes de cadena opcionales:

```rust
StatementInner::Dim { decls } => {
    bytes.extend_from_slice(Keyword::Dim.internal_code().to_be_bytes().as_slice());
    
    for (i, decl) in decls.iter().enumerate() {
        match decl {
            DimInner::DimInner1D { identifier, size, string_length, .. } => {
                identifier.write_bytes(bytes);
                bytes.push(b'(');
                size.write_bytes(bytes, preserve_source_wording);
                bytes.push(b')');
                
                if let Some(len) = string_length {
                    bytes.push(b'*');
                    len.write_bytes(bytes, preserve_source_wording);
                }
            }
            DimInner::DimInner2D { identifier, rows, cols, string_length, .. } => {
                identifier.write_bytes(bytes);
                bytes.push(b'(');
                rows.write_bytes(bytes, preserve_source_wording);
                bytes.push(b',');
                cols.write_bytes(bytes, preserve_source_wording);
                bytes.push(b')');
                
                if let Some(len) = string_length {
                    bytes.push(b'*');
                    len.write_bytes(bytes, preserve_source_wording);
                }
            }
        }
        
        if i < decls.len() - 1 {
            bytes.push(b',');
        }
    }
}
```

Ejemplo 1:

`DIM B$(5)*20`

Desglose hexadecimal :
- `F1 88` = DIM keyword (0xF188)
- `42` = 'B'
- `24` = '$'
- `28` = '('
- `35` = '5'
- `29` = ')'
- `2A` = '*'
- `32 30` = "20"

Ejemplo 2:

`DIM C(3,4)`

Desglose hexadecimal:
- `F1 88` = DIM keyword (0xF188)
- `43` = 'C'
- `28` = '('
- `33` = '3'
- `2C` = ','
- `34` = '4'
- `29` = ')'

5.5 Sentencias de Control de Flujo Avanzadas (*ON..GOTO* y *ON..GOSUB*)

```rust
StatementInner::OnGoto { expr, targets } => {
    bytes.extend_from_slice(Keyword::On.internal_code().to_be_bytes().as_slice());
    expr.write_bytes(bytes, preserve_source_wording);
    bytes.extend_from_slice(Keyword::Goto.internal_code().to_be_bytes().as_slice());
    for (i, target) in targets.iter().enumerate() {
        target.write_bytes(bytes, preserve_source_wording);
        if i < targets.len() - 1 {
            bytes.push(b',');
        }
    }
}
```

Ejemplos:
- `ON X GOTO 100,200,300` → `F1 9C 58 F1 92 31 30 30 2C 32 30 30 2C 33 30 30`
- `ON ERROR GOTO 999` → `F1 9C F1 8F F1 92 39 39 39`


6. Generación de Expresiones

6.1 Manejo de Precedencia y Paréntesis

El sistema de precedencia *siempre* sigue el orden de izquierda a derecha:


```rust
pub fn write_bytes_with_context_and_parens(
    &self,
    bytes: &mut Vec<u8>,
    parent_prec: u8,
    is_right_side: bool,
    preserve_source_wording: bool,
) {
    match self {
        ExprInner::Binary(left, op, right) => {
            let my_prec = op.precedence();
            let needs_parens = !preserve_source_wording
                && (my_prec < parent_prec || (my_prec == parent_prec && is_right_side));
            
            if needs_parens {
                bytes.push(b'(');
            }
            
            left.write_bytes_with_context_and_parens(bytes, my_prec, false, preserve_source_wording);
            op.write_bytes(bytes);
            right.write_bytes_with_context_and_parens(bytes, my_prec, true, preserve_source_wording);
            
            if needs_parens {
                bytes.push(b')');
            }
        }
        // ... otros casos
    }
}
```

6.1.1 Tabla de Precedencias (de mayor a menor):

```rust
impl BinaryOp {
    pub fn precedence(&self) -> u8 {
        match self {
            BinaryOp::Or => 1,         // Más baja
            BinaryOp::And => 2,
            BinaryOp::Eq | BinaryOp::Neq | BinaryOp::Lt | 
            BinaryOp::Gt | BinaryOp::Leq | BinaryOp::Geq => 3,
            BinaryOp::Add | BinaryOp::Sub => 4,
            BinaryOp::Mul | BinaryOp::Div => 5,
            BinaryOp::Exp => 6,        // Más alta
        }
    }
}
```

6.2 Operadores

Operadores Unarios:
```rust
impl UnaryOp {
    pub fn write_bytes(&self, bytes: &mut Vec<u8>) {
        match self {
            UnaryOp::Plus => bytes.push(b'+'),
            UnaryOp::Minus => bytes.push(b'-'),
            UnaryOp::Not => bytes.extend_from_slice(Keyword::Not.internal_code().to_be_bytes().as_slice()),
        }
    }
}
```
Ejemplo:

- `+5` → `2B 35` ('+' + '5')
- `-10` → `2D 31 30` ('-' + "10")
- `NOT X` → `F1 9F 58` (NOT keyword 0xF19F + 'X')

Operadores Binarios:

```rust
impl BinaryOp {
    pub fn write_bytes(&self, bytes: &mut Vec<u8>) {
        match self {
            BinaryOp::Add => bytes.push(b'+'),
            BinaryOp::Sub => bytes.push(b'-'),
            BinaryOp::Mul => bytes.push(b'*'),
            BinaryOp::Div => bytes.push(b'/'),
            BinaryOp::Exp => bytes.push(b'^'),
            BinaryOp::Eq => bytes.push(b'='),
            BinaryOp::Neq => bytes.extend_from_slice(b"<>"),
            BinaryOp::Lt => bytes.push(b'<'),
            BinaryOp::Gt => bytes.push(b'>'),
            BinaryOp::Leq => bytes.extend_from_slice(b"<="),
            BinaryOp::Geq => bytes.extend_from_slice(b">="),
            BinaryOp::And => bytes.extend_from_slice(Keyword::And.internal_code().to_be_bytes().as_slice()),
            BinaryOp::Or => bytes.extend_from_slice(Keyword::Or.internal_code().to_be_bytes().as_slice()),
        }
    }
}
```
Tabla de operadores binarios:

| Operador | Código Fuente | Bytecode (hex) | ASCII |
|----------|---------------|----------------|-------|
| Suma | `A + B` | `41 2B 42` | "A+B" |
| Resta | `A - B` | `41 2D 42` | "A-B" |
| Multiplicación | `A * B` | `41 2A 42` | "A*B" |
| División | `A / B` | `41 2F 42` | "A/B" |
| Exponenciación | `A ^ B` | `41 5E 42` | "A^B" |
| Igual | `A = B` | `41 3D 42` | "A=B" |
| Diferente | `A <> B` | `41 3C 3E 42` | "A<>B" |
| Menor | `A < B` | `41 3C 42` | "A<B" |
| Mayor | `A > B` | `41 3E 42` | "A>B" |
| Menor o igual | `A <= B` | `41 3C 3D 42` | "A<=B" |
| Mayor o igual | `A >= B` | `41 3E 3D 42` | "A>=B" |
| Y lógico | `A AND B` | `41 F1 7D 42` | "A" + AND(0xF17D) + "B" |
| O lógico | `A OR B` | `41 F1 A0 42` | "A" + OR(0xF1A0) + "B" |


Ejemplo:

`X > 0 AND Y < 10`

Desglose hexadecimal:

- `58` = 'X'
- `3E` = '>'
- `30` = '0'
- `F1 7D` = AND keyword (0xF17D)
- `59` = 'Y'
- `3C` = '<'
- `31 30` = "10"


6.3 Funciones Built-in

Las funciones integradas requieren manejo especial de paréntesis y argumentos múltiples:

```rust
impl Function {
    pub fn write_bytes_preserving_source_parens(&self, bytes: &mut Vec<u8>, preserve_source_wording: bool) {
        match &self.inner {
            FunctionInner::MidDollar { string, start, length } => {
                bytes.extend_from_slice(Keyword::MidDollar.internal_code().to_be_bytes().as_slice());
                bytes.push(b'(');
                string.write_bytes(bytes, preserve_source_wording);
                bytes.push(b',');
                start.write_bytes(bytes, preserve_source_wording);
                bytes.push(b',');
                length.write_bytes(bytes, preserve_source_wording);
                bytes.push(b')');
            }
            FunctionInner::Sin { expr } => {
                bytes.extend_from_slice(Keyword::Sin.internal_code().to_be_bytes().as_slice());
                expr.write_bytes_with_context_and_parens(bytes, 20, false, preserve_source_wording);
            }
            // ... más funciones
        }
    }
}
```

6.3.1 Funciones Matemáticas:
```rust
// Función con paréntesis obligatorios
FunctionInner::Sin { expr } => {
    bytes.extend_from_slice(Keyword::Sin.internal_code().to_be_bytes().as_slice());
    expr.write_bytes_with_context_and_parens(bytes, 20, false, preserve_source_wording);
}
```

| Función | Código Fuente | Bytecode (hex) | Descripción |
|---------|---------------|----------------|-------------|
| `SIN(X)` | `SIN(X)` | `F1 AD 58` | Seno de X |
| `COS(Y)` | `COS(Y)` | `F1 7F 59` | Coseno de Y |
| `TAN(Z)` | `TAN(Z)` | `F1 AB 5A` | Tangente de Z |
| `SQR(A)` | `SQR(A)` | `F1 A7 41` | Raíz cuadrada de A |
| `INT(B)` | `INT(B)` | `F1 94 42` | Parte entera de B |
| `ABS(C)` | `ABS(C)` | `F1 76 43` | Valor absoluto de C |

6.3.2 Funciones de Cadena:
```rust
// Funciones con múltiples argumentos requieren paréntesis y comas
FunctionInner::MidDollar { string, start, length } => {
    bytes.extend_from_slice(Keyword::MidDollar.internal_code().to_be_bytes().as_slice());
    bytes.push(b'(');
    string.write_bytes(bytes, preserve_source_wording);
    bytes.push(b',');
    start.write_bytes(bytes, preserve_source_wording);
    bytes.push(b',');
    length.write_bytes(bytes, preserve_source_wording);
    bytes.push(b')');
}
```

| Función | Código Fuente | Bytecode (hex y ASCII) | Descripción |
|---------|---------------|------------------------|-------------|
| `MID$(S$,2,3)` | `MID$(S$,2,3)` | `F1 99 28 53 24 2C 32 2C 33 29` | Subcadena |
| `LEFT$(A$,5)` | `LEFT$(A$,5)` | `F1 97 28 41 24 2C 35 29` | Primeros N caracteres |
| `RIGHT$(B$,3)` | `RIGHT$(B$,3)` | `F1 A4 28 42 24 2C 33 29` | Últimos N caracteres |
| `LEN(X$)` | `LEN(X$)` | `F1 98 58 24` | Longitud de cadena |
| `ASC(C$)` | `ASC(C$)` | `F1 77 43 24` | Código ASCII |
| `CHR$(65)` | `CHR$(65)` | `F1 7E 36 35` | Carácter desde código |

Ejemplo:
 `MID$(NAME$, 2, 4)`

Desglose hexadecimal:
```
F1 99    = MID$ keyword (0xF199)
28       = '('
4E 41 4D 45 24 = "NAME$"
2C       = ','
32       = '2'
2C       = ','
34       = '4'
29       = ')'
```

7. L-Values (Referencias de Variables)

Los L-Values representan ubicaciones de memoria que pueden ser asignadas, incluyendo variables simples, arrays y áreas de memoria especiales:

7.1 Variables Simples y Arrays

```rust
impl LValue {
    pub fn write_bytes(&self, bytes: &mut Vec<u8>, preserve_source_wording: bool) {
        match &self.inner {
            LValueInner::Identifier(id) => {
                id.write_bytes(bytes);
            }
            LValueInner::BuiltInIdentifier(bi) => {
                bytes.extend_from_slice(bi.internal_code().to_be_bytes().as_slice());
            }
            LValueInner::Array1DAccess { identifier, index } => {
                identifier.write_bytes(bytes);
                bytes.push(b'(');
                index.write_bytes(bytes, preserve_source_wording);
                bytes.push(b')');
            }
            LValueInner::Array2DAccess { identifier, row_index, col_index } => {
                identifier.write_bytes(bytes);
                bytes.push(b'(');
                row_index.write_bytes(bytes, preserve_source_wording);
                bytes.push(b',');
                col_index.write_bytes(bytes, preserve_source_wording);
                bytes.push(b')');
            }
            LValueInner::FixedMemoryAreaAccess { index, has_dollar } => {
                bytes.push(b'@');
                if *has_dollar {
                    bytes.push(b'$');
                }
                bytes.push(b'(');
                index.write_bytes(bytes, preserve_source_wording);
                bytes.push(b')');
            }
        }
    }
}
```

Variables Numéricas:
```rust
LValueInner::Identifier(id) => {
    id.write_bytes(bytes);  // Directo a ASCII
}
```

| Variable | Bytecode (hex) | ASCII |
|----------|----------------|-------|
| `A` | `41` | "A" |
| `B1` | `42 31` | "B1" |
| `XY` | `58 59` | "XY" |

Variables String:
```rust
// El símbolo $ se incluye en el identificador
Identifier { char1: 'X', char2: None, has_dollar: true }
```

| Variable | Bytecode (hex) | ASCII |
|----------|----------------|-------|
| `A$` | `41 24` | "A$" |
| `NAME$` | `4E 41 4D 45 24` | "NAME$" |
| `X1$` | `58 31 24` | "X1$" |

Acceso a Arrays 

```rust
LValueInner::Array1DAccess { identifier, index } => {
    identifier.write_bytes(bytes);
    bytes.push(b'(');
    index.write_bytes(bytes, preserve_source_wording);
    bytes.push(b')');
}
```

Ejemplos:

| Código Fuente | Bytecode (hex y ASCII) | Descripción |
|---------------|------------------------|-------------|
| `A(5)` | `41 28 35 29` | A + '(' + '5' + ')' |
| `B$(I)` | `42 24 28 49 29` | B$ + '(' + I + ')' |
| `DATA(X+1)` | `44 41 54 41 28 58 2B 31 29` | Array con expresión |

Ejemplo : 

`SCORES(PLAYER)`

Desglose hexadecimal:
```
53 43 4F 52 45 53 = "SCORES"
28                = '('
50 4C 41 59 45 52 = "PLAYER"
29                = ')'
```

Acceso a Áreas de Memoria Fijas

El Sharp PC-1500 permite acceso directo a áreas de memoria específicas:

```rust
LValueInner::FixedMemoryAreaAccess { index, has_dollar } => {
    bytes.push(b'@');
    if *has_dollar {
        bytes.push(b'$');
    }
    bytes.push(b'(');
    index.write_bytes(bytes, preserve_source_wording);
    bytes.push(b')');
}
```

Ejemplos de acceso a memoria:

| Código Fuente | Bytecode (hex y ASCII) | Descripción |
|---------------|------------------------|-------------|
| `@(100)` | `40 28 31 30 30 29` | Memoria numérica en dirección 100 |
| `@$(200)` | `40 24 28 32 30 30 29` | Memoria string en dirección 200 |
| `@(BASE+OFFSET)` | `40 28 42 41 53 45 2B 4F 46 46 53 45 54 29` | Con expresión |

Ejemplo:

 `@$(ADDR)`

Desglose hexadecimal:
```
40    = '@'
24    = '$'
28    = '('
41 44 44 52 = "ADDR"
29    = ')'
```

Existen una serie de restricciones a la hora de definir los identificadores:

- Máximo 2 caracteres alfanuméricos
- Primer carácter debe ser letra (A-Z)
- Segundo carácter puede ser letra o dígito

8. Integración desde *main.rs*

```rust
// Generar bytes del programa
let mut program_bytes = Vec::new();
program.write_bytes(&mut program_bytes, args.preserve_source_wording);

// Crear bytes de salida con o sin header
let output_bytes = if args.no_header {
    program_bytes
} else {
    let header = Header::new(&program_name, program_bytes.len() as u16);
    let mut output_bytes = header.to_bytes();
    output_bytes.extend(program_bytes);
    output_bytes
};

// Escribir archivo final
let output_path = args.output.unwrap_or_else(|| PathBuf::from("a.bin"));
fs::write(&output_path, &output_bytes)?;
```

**9. Ejemplo Completo de Compilación**

**9.1 Código Fuente BASIC**

```basic
10 "DEMO" FOR I = 1 TO 10
20 PRINT "Hello"; I
30 NEXT I
40 END
```

**9.2 Bytecode Resultante (Hexadecimal)**

**Header**:
```
01                    // Magic number
40 43 4F 4D          // "@COM"
44 45 4D 4F 00 00 00 00 00 00 00 00 00 00 00 00  // "DEMO" + padding
00 C5                 // Start address
00 2F                 // Capacity (program_length - 1)
04 4F                 // Entry address
```

**Programa**:
```
00 0A                 // Line number 10
0F                    // Line size
22 44 45 4D 4F 22     // "DEMO"
F1 A5                 // FOR keyword
49                    // I
3D                    // =
31                    // 1
F1 B1                 // TO keyword
31 30                 // 10
0D                    // Carriage return

00 14                 // Line number 20
0B                    // Line size
F0 97                 // PRINT keyword
22 48 65 6C 6C 6F 22  // "Hello"
3B                    // ;
49                    // I
0D                    // Carriage return

00 1E                 // Line number 30
04                    // Line size
F1 9A                 // NEXT keyword
49                    // I
0D                    // Carriage return

00 28                 // Line number 40
03                    // Line size
F1 8E                 // END keyword
0D                    // Carriage return

FF                    // End of program marker
```



