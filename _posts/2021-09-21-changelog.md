---
title: "Changelog as of September 21, 2021"
date: 2021-09-22 16:41:55 +/-0800
categories: [Updates, "Changelogs"]
tags: [updates,'initial']     # TAG names should always be lowercase
---

# Release Highlights
- **First update changelog!**
- **Added** MakeFiles and updated setup.cfgs to easily compile and build grammer from qasm3.g4
- **Initial translator package** with limited functionality.
- **Currently supported definitions from ast**: BranchingStatement, ClassicalAssignment, ClassicalDeclaration, ConstantDeclaration, ForInLoop, QuantumBarrier, QuantumGate, QuantumGateDefinition, QuantumMeasurementAssignment, QuantumPhase, QuantumReset, QubitDeclaration, Statement, Concatenation, Identifier, IndexIdentifier, RangeDefinition, Selection, Slice, Subscript, BinaryExpression, BooleanLiteral, Constant, DurationLiteral, Expression, FunctionCall, Identifier, IndexExpression, IntegerLiteral, RealLiteral, StringLiteral, UnaryExpression, Span.


## Update notes
- **Added** MakeFile's to source/openqasm and source/grammer to compile and build grammer from qasm3.g4
- **Updated** setup.cfgs with pylint and black support
- Update log <br />
>  1. **source/openqasm/tests/build_ast.py -**  Test parser for validating statement numbers, OpenQASM version and ForLoop Listing for example adder.qasm in openqasm/examples    
>   
>  2. **source/openqasm/src/openqasm/translator/context.py -** Stores a parseable context in a dict-like structure.
        * **_OpenQASMIdentifier:** Class to store necessary information about location of identifiers defined by type Span from ast w.r.t source. Keeps track of  start and end of line following antlr convention.
        * **class OpenQASMContext:** Stores the necessary content to parse the OpenQASM 3.0 file
            * **add_symbol** = Adds an initialized symbol. Stores in _symbols dict
            * **lookup** = searches value in context dict _symbols
            * **declare_symbol** = Adds uninitialized symbol with None value to context dict _symbols  
>  3. **source/openqasm/src/openqasm/translator/exceptions.py -** Handles exceptions for the whole translator package. Also holds location info for each error occurred w.r.t source.  
**Methods:**  
        * **UnsupportedFeature =** raises error and displays unsupported feature_name with a reason
        * **UnsupportedExpressionType =** raises an invalid expression_type_name error.
        * **UndefinedSymbol =** raises undeclared identifier error
        * **UnkownConstant =** raises undeclared constant identifier error
        * **UninitializedSymbol =** raises error for an uninitialized identifier which requires an initialization
        * **MissingExpression =**  Raises error for a wrongly structured expression or missing expression
>  4. **source/openqasm/src/openqasm/translator/expressions.py -**  Computes Expression according to type. Main class = _ComputeExpressionNamespace  
**Methods:**  
        * **compute_BinaryExpression =** Computes lhs and rhs w.r.t context grammar. Returns evaluated expression using default eval func
        * **compute_UnaryExpression =** Computes value of Unary expression with the operator w.r.t context grammar. Returns evaluated expression using default eval func
        * **compute_Constant =** Checks for constant values in supported list and returns the constant value corresponding to _CONSTANT_VALUES dict
        * **compute_Identifier =** Returns Identifier corresponding to the context stored in _symbol dict in context.py
        * **compute_IntegerLiteral =** Returns Integer Literal corresponding to the context stored in _symbol dict in context.py
        * **compute_RealLiteral =** Returns Real Literal corresponding to the context stored in _symbol dict in context.py
        * **compute_BooleanLiteral =** Returns Boolean literal corresponding to the context stored in _symbol dict in context.py
        * **compute_StringLiteral =** Returns Boolean literal corresponding to the context stored in _symbol dict in context.py
        * **compute_DurationLiteral =**  Currently unsupported. Throws a UnsupportedExpressionType error.
        * **compute_FunctionCall =** Compute_expression for each of the passed list of arguments w.r.t context grammar and returns values of function name corresponding to the context stored in _symbol dict in context.py
        * **compute_IndexExpression =** Compute_expression for index value w.r.t context grammar and returns compute_expression for the expression part for the particular index passed.
        * **compute_Expression =** Computes generic expression; Stores method_name as a formatted string with prefix compute_(input_expression_type)  and returns attribute of the method w.r.t. Context grammar passing the relevant expression and context. Throws UnsupportedExpressionType exception if expression not currently supported by context.
>  5. **source/openqasm/src/openqasm/translator/identifiers.py -**  Defines identifiers and processes the relevant type of identifier as per definition in ast.  
**Methods:**
        * **get_identifier =** takes in either an Identifier or IndexIdentifier from ast and returns attributes after passing it to the relevant method_name in the _IdentifierRetrieverNamespace
        * **get_Subscript =** computes the index and returns lookup value in context _symbols  dict for that index.
        * **get_Selection =** computes list of input indexes and returns list of lookup symbols w.r.t context
        * **get_Slice =**  computes the input range and returns list of range of objects returned after evaluating, start, end and step points from context definition
        * **get_Concatenation =**  computes concatenated list of passed arrays w.r.t context. Checks if the returned identifier lookup is a list and casts it to a list if needed.
>  6. **source/openqasm/src/openqasm/translator/modifiers.py -**  Defines apply_modifier to apply  appropriate modifier to quantum gate. Supports inv, pow and ctrl. .negctrl type also supported as of [2c5bc5]. Also compute_expression for number of ctrl and power as per context.  
>   
>  7. **source/openqasm/src/openqasm/translator/translator.py -**  Translator to convert OpenQASM 3.0 into a qiskit QuantumCircuit instance  
**Methods:**
        * **translate =** Translates given ast to a qiskit QuantumCircuit instance. Declares a QuantumCircuit and context object and passes on each ast statement to _process_Statement for further breaking it down and retrieves the resulting processed qiskit QuantumCircuit object and returns the same.
        * **_process_statement =** Extracts each statement type, name and formats function name as per its relevant function name to be passed on to for relevant processing. The relevant processing_function_name is passed on with statement, circuit and context as arguments. Also checks if the function name is a supported feature and returns an UnsupportedFeature exception
        * **_supported_features =** Returns list of supported AST type names
        * **_process_QubitDeclaration =** Process QubitDeclaration node in AST. Sets default size to 1 otherwise compute_expression as per context for quantum_register_size. Declares the register and adds it to the circuit as well as adds qubit.name and register to _symbol dict in context.
        * **_process_ConstantDeclaration =** Process ConstantDeclaration node in AST. Checks current statement node and either declares it as None if uninitialised or adds it to _symbol dict in context
        * **_process_ClassicalDeclaration =** Processes ClassicalDeclaration node in AST if supported. Initializes expression to none or adds it to _symbol dict.
        * **_process_QuantumReset =** Processes QuantumReset node and applies reset operation to the given qubit in context
        * **_process_QuantumGate =** Processes QuantumGate node in AST. Appends relevant gate operation to the qubit in context. Applies the relevant gate modifier from modifiers.py if applicable
        * **_process_QuantumGateDefinition =** Processes custom parameterized gates as specified in QuantumGateDefinition from ast. Returns an equivalent custom gate to the input definition.
        * **_process_QuantumPhase =** Processes a custom phase input to a gate upto a global phase. Appends the equivalent phased_gate to the circuit object declared in translate method
