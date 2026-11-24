# Rule catalog

Every constraint of the profile, extracted from `../profile/PDDL-SysML_Profile.mdzip`.
The full OCL expressions are in [`rules.csv`](rules.csv).

| Severity | Count | Meaning |
|---|---|---|
| `error` | 70 | The annotation is not well-formed. The generated PDDL would be invalid or would not say what the annotation means. |
| `warning` | 13 | The annotation is incomplete but can still be processed. |
| `info` | 1 | Points out a known limitation. |

84 rules in total.

Names in PDDL are global within a domain or a problem, and planners compare them without case. The rules
on unique names therefore ignore case and package boundaries.

## Example findings

On `V3` from the [`mps500-planning-example`](https://github.com/HamiedNabizada/mps500-planning-example) repository, a variant with three injected
errors, the suite reports these three and nothing else:

    Element  Severity  Rule     Message
    f17      error     FML-02   'or' has fewer than two operands. Either add operands
                                or use the single operand directly.
    f29      error     FML-04   'imply' needs both a condition and a consequence.
    f32      error     FMA-03   The literal has no numeric value. Set 'value'.

A disjunction with a single operand, as in `FML-02`, is valid PDDL syntax, so a planner would
not report it.

## Standard OCL

The OCL in [`rules.csv`](rules.csv) is written for the evaluation in MSoSA. There, `self` is the annotated element itself, and `<Stereotype>.allInstances()` yields annotated elements. [`rules_standard_ocl.csv`](rules_standard_ocl.csv) gives every rule a second time, with the same ID, following the UML semantics of profiles:

- `self` is the stereotype application. A rule reaches the annotated element through `base_<Metaclass>`, for example `self.base_Class.name`.
- Tags typed by a stereotype hold stereotype applications, and `<Stereotype>.allInstances()` yields stereotype applications.
- The expressions use only the OCL 2.4 standard library, for example `toLowerCase()` instead of `toLower()`, and `characters()` and `indexOf()` instead of regular expressions.
- An optional tag without a value is `null`.
- Where a rule asks whether `closure` reaches an element, the start elements are tested explicitly, because tools differ in whether `closure` includes them.

47 of the 84 rules differ from the MSoSA version. The other 37 are identical.

The standard version was evaluated outside MSoSA with Eclipse OCL on an Eclipse UML2 export of the models. On the 22 models of the [`mps500-planning-example`](https://github.com/HamiedNabizada/mps500-planning-example) repository it gives the same verdict as MSoSA for each of the 19,984 pairs of element and rule. On further test models with injected errors, 82 rules report at the same elements in both versions.

## Domain (`DOM`, 6 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `DOM-01` | `PDDL_Domain` | error | The domain has a name. PDDL requires named domains. |
| `DOM-02` | `PDDL_Domain` | error | Checks that domain names are unique across packages, ignoring case. |
| `DOM-03` | `PDDL_Domain` | warning | The model contains at least one PDDL_Type. The check covers the whole model, because the types may live in a different package than the domain. |
| `DOM-04` | `PDDL_Domain` | error | The domain name is a valid PDDL identifier and not a reserved PDDL keyword. |
| `DOM-05` | `PDDL_Constant` | error | The constant name is a valid PDDL identifier and not a reserved PDDL keyword. |
| `DOM-06` | `PDDL_Constant` | error | Checks that constant names are unique and differ from all object names, ignoring case, because constants and objects share one PDDL namespace and planners read names without case. |

## Types (`TYP`, 5 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `TYP-01` | `PDDL_Type` | error | Checks that type names are unique across packages, ignoring case (the transformation also writes type names in lower case). |
| `TYP-02` | `PDDL_Type` | warning | PDDL_Type is applied to a SysML Block rather than to a plain class. |
| `TYP-03` | `PDDL_Type` | error | The type name is a valid PDDL identifier (a letter first, then letters, digits, '_' or '-') and not a reserved PDDL keyword. |
| `TYP-04` | `PDDL_Type` | error | Checks that the type hierarchy has no cycle: a type is not among the ancestors of its supertypes, followed over all generalizations and over parentTypeRef, without a depth limit. |
| `TYP-05` | `PDDL_Type` | warning | Checks that a type has at most one generalization. A PDDL type has one supertype, and the transformation writes the first generalization of the classifier as supertype; further generalizations would be lost without notice. |

## Predicates (`PRD`, 5 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `PRD-01` | `PDDL_Predicate` | error | The predicate has a name. |
| `PRD-02` | `PDDL_Predicate` | error | Every parameter of the predicate is typed with a PDDL_Type. |
| `PRD-03` | `PDDL_Predicate` | error | Checks that predicate names are unique across packages, ignoring case. |
| `PRD-04` | `PDDL_Predicate` | error | The predicate name is a valid PDDL identifier and not a reserved PDDL keyword. |
| `PRD-05` | `PDDL_Predicate` | warning | Checks that a predicate required in a precondition has a source: an action effect that adds it, or an initial state that sets it, either by a PDDL_InitState dependency that is not negated or by an object slot whose owner and value match the two parameter types (as the problem template derives binary facts). Read are the input pins of Mode A actions and the atoms of Mode B preconditions reachable through 'and', 'or', the body of 'exists' and 'forall' and the consequence of 'imply'. A negated precondition needs no source and is not checked; negation and preservation are read from pin annotations and object flows. Sources are output pins of Mode A actions that are not negated and atoms of Mode B effects reachable through 'and', the effect of a conditional effect and the body of 'forall'. Not read: the conditions of conditional effects and of 'imply', and goals. The check covers all problems of the model together. Warning: a model may leave the source out on purpose. |

## Functions (`FUN`, 3 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `FUN-01` | `PDDL_Function` | error | The function has a name. |
| `FUN-02` | `PDDL_Function` | error | Checks that function names are unique across packages, ignoring case. |
| `FUN-03` | `PDDL_Function` | error | The function name is a valid PDDL identifier and not a reserved PDDL keyword. 'total-cost' is allowed, because PDDL uses it for action costs. |

## Actions (`ACN`, 11 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `ACN-01` | `PDDL_Action` | error | The action has an actionName. |
| `ACN-02` | `PDDL_Action` | warning | Checks that an action changes something. Mode B (the tag 'precondition' or 'effect' is set): the tag 'effect' is set. Mode A: the action has an output pin typed by a predicate or a function, or an input predicate pin that is neither preserved nor negated, which the negation rule deletes. This includes an action without any precondition or effect. The tag 'cost' alone does not count as an effect, because it changes the plan cost, not the state. |
| `ACN-03` | `PDDL_DurativeAction` | error | The duration of a durative action is a positive number. |
| `ACN-04` | `PDDL_Action` | error | Checks that action names are unique, ignoring case, durative actions included. Two actions may share a name only if they call the same behavior. |
| `ACN-05` | `PDDL_Action` | error | Every input pin of the action is untyped or typed with a PDDL_Type, a PDDL_Predicate or a PDDL_Function. Predicate- and function-typed pins carry the Mode A precondition and effect and are not parameters. Whether a function-typed pin states its role is checked by ACN-10. |
| `ACN-06` | `PDDL_Action` | error | The action cost is not negative. |
| `ACN-07` | `PDDL_Action` | error | The action name is a valid PDDL identifier and not a reserved PDDL keyword. |
| `ACN-08` | `PDDL_DurativeAction` | info | Durative actions (PDDL 2.1 Level 3) can be specified with the profile, but the templates do not generate them. |
| `ACN-09` | `PDDL_Action` | warning | The tag 'parameters' is not set when typed pins exist. The transformation prefers the pins and uses the tag only for actions without typed pins. |
| `ACN-10` | `PDDL_Action` | error | In Mode A, every pin typed with a PDDL_Function carries a PDDL_PinAnnotation that states its role: 'comparison' on an input pin (precondition), 'numericEffect' on an output pin (effect). Without it the template cannot transform the pin. Mode B actions (precondition or effect set) are exempt. |
| `ACN-11` | `PDDL_Action` | warning | Checks the predicate pins of one call for contradicting atoms: an output pin sets an atom that a negated output pin deletes, an input pin preserves an atom that a negated output pin deletes, or the same atom is preserved on one input pin and not on another. An atom is the predicate of the pin together with its tag 'arguments'; pins without 'arguments' count as the same atom. Only pin annotations are read, not object flows. PDDL gives each case a defined meaning (delete before add), but the annotation contradicts itself. |

## Pins (`PIN`, 8 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `PIN-01` | `PDDL_PinAnnotation` | error | A pin annotation is not both negated and preserved. The two roles contradict each other. |
| `PIN-02` | `PDDL_PinAnnotation` | warning | A pin annotation and the annotation on the connected object flow agree. If they disagree, the transformation reads the pin and ignores the flow. |
| `PIN-03` | `PDDL_PinAnnotation` | error | Numeric tags are set only on pins typed with a PDDL_Function. Such pins carry neither isNegated nor isPreserved, which belong to predicate pins. |
| `PIN-04` | `PDDL_PinAnnotation` | error | 'comparison' is set only on input pins and 'numericEffect' only on output pins. |
| `PIN-05` | `PDDL_PinAnnotation` | error | A comparison or numeric effect on a pin has exactly one right-hand side, numericValue or numericOperand. Neither is set without a comparison or numeric effect. |
| `PIN-06` | `PDDL_PinAnnotation` | error | numericOperand is a numeric term (FunctionAtom, Literal or ArithmeticExpression). |
| `PIN-07` | `PDDL_PinAnnotation` | error | Checks that 'arguments' is set only on predicate-typed pins and has one entry per parameter of the predicate. Every entry counts, including a repeated one such as '?c ?c'. A wrong count gives an atom of the wrong arity. |
| `PIN-08` | `PDDL_PinAnnotation` | error | Every entry of 'arguments' names a parameter pin (typed with a PDDL_Type) of the same action, with or without a leading '?'. An output pin 'xOut' counts as parameter 'x', as in the domain template. |

## Object flows (`FLW`, 1 rule)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `FLW-01` | `PDDL_FlowAnnotation` | error | A flow annotation is not both negated and preserved. The two roles contradict each other. |

## Problem, objects, init and goal (`PRB`, 20 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `PRB-01` | `PDDL_Problem` | error | The problem references a domain. |
| `PRB-02` | `PDDL_Object` | error | The object has a classifier (type). |
| `PRB-03` | `PDDL_InitState` | error | An init state references a predicate or a function. |
| `PRB-04` | `PDDL_GoalState` | error | A goal state references a predicate or a function. |
| `PRB-05` | `PDDL_InitState` | error | An init state does not set predicateRef and functionRef at the same time. PRB-03 reports the case that neither is set. |
| `PRB-06` | `PDDL_GoalState` | error | A goal state does not set predicateRef and functionRef at the same time. PRB-04 reports the case that neither is set. |
| `PRB-07` | `PDDL_InitState` | error | An init state with functionRef carries a value. |
| `PRB-08` | `PDDL_GoalState` | warning | A numeric goal (functionRef) carries a target value. |
| `PRB-09` | `PDDL_InitState` | warning | Checks that additionalArgs are set only together with a predicateRef or a functionRef. Arguments without a reference have no effect. |
| `PRB-10` | `PDDL_Problem` | error | A metricExpression comes with a metricDirection (minimize or maximize). |
| `PRB-11` | `PDDL_Problem` | error | The problem name is a valid PDDL identifier and not a reserved PDDL keyword. |
| `PRB-12` | `PDDL_Object` | error | The object name is a valid PDDL identifier and not a reserved PDDL keyword. |
| `PRB-13` | `PDDL_Object` | error | Checks that object names are unique in the model, ignoring case. Only objects of different problems may share a name. |
| `PRB-14` | `PDDL_Object` | warning | The tag 'objectType' matches the classifier of the instance specification. If it does not, the transformation uses the classifier. |
| `PRB-15` | `PDDL_InitState` | error | Checks that an init state has as many arguments as its predicate or function has parameters. The arguments are the clients of the dependency followed by 'additionalArgs'. A package as client, such as the problem package as the formal client of a fact without arguments, does not count; the transformation ignores it as well. The same object in two argument positions needs 'additionalArgs', because the clients of a dependency form a set. |
| `PRB-16` | `PDDL_GoalState` | error | Checks that a goal state has as many arguments as its predicate or function has parameters. The arguments are the clients of the dependency followed by 'additionalArgs'; a goal over a type has the type as its one client. A package as client, such as the problem package as the formal client of a fact without arguments, does not count; the transformation ignores it as well. The same object in two argument positions needs 'additionalArgs', because the clients of a dependency form a set. |
| `PRB-17` | `PDDL_Object` | error | The classifier of a PDDL_Object is a PDDL_Type. Otherwise the object would be typed with something the (:types ...) section never declares. |
| `PRB-18` | `PDDL_Problem` | error | The 'domain' tag points to a package with PDDL_Domain. PRB-01 only checks that the tag is set. |
| `PRB-19` | `PDDL_InitState` | warning | Checks that the reference of an init state ('predicateRef' or 'functionRef') is the supplier of its dependency. The transformation takes the predicate or function from the supplier, the rules PRB-03 to PRB-15 read the reference; if the two differ, the rules check a different fact from the one generated. |
| `PRB-20` | `PDDL_GoalState` | warning | Checks that the reference of a goal state ('predicateRef' or 'functionRef') is the supplier of its dependency. The transformation takes the predicate or function from the supplier, the rules PRB-04 to PRB-16 read the reference; if the two differ, the rules check a different fact from the one generated. |

## Logical formulas (`FML`, 6 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `FML-01` | `PDDL_And` | error | A conjunction ('and') has at least two operands. |
| `FML-02` | `PDDL_Or` | error | A disjunction ('or') has at least two operands. |
| `FML-03` | `PDDL_Not` | error | A negation ('not') has exactly one operand. |
| `FML-04` | `PDDL_Imply` | error | An implication ('imply') has a condition and a consequence. |
| `FML-05` | `PDDL_Or` | error | Checks that 'or' does not appear in an action effect (PDDL allows disjunction only in conditions). Effect positions are the effect itself and everything reachable from it through 'and', the effect of a conditional effect and the body of a 'forall', at any depth; the condition of a conditional effect is not an effect position. |
| `FML-06` | `PDDL_Not` | error | Checks that 'not' in an action effect negates a predicate atom only. Effect positions are the effect itself and everything reachable from it through 'and', the effect of a conditional effect and the body of a 'forall', at any depth; the condition of a conditional effect is not an effect position. |

## Quantifiers (`FMQ`, 2 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `FMQ-01` | `PDDL_Exists` | error | An existential quantifier ('exists') has a scope and at least one variable. |
| `FMQ-02` | `PDDL_Forall` | error | A universal quantifier ('forall') has a scope and at least one variable. |

## Atoms and literals (`FMA`, 7 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `FMA-01` | `PDDL_PredicateAtom` | error | A predicate atom references a predicate. |
| `FMA-02` | `PDDL_FunctionAtom` | error | A function atom references a function. |
| `FMA-03` | `PDDL_Literal` | error | A literal has a numeric value. |
| `FMA-04` | `PDDL_PredicateAtom` | error | Checks that a predicate atom in a formula (Mode B) has as many 'arguments' as its predicate has parameters. |
| `FMA-05` | `PDDL_FunctionAtom` | error | Checks that a function atom in a formula (Mode B) has as many 'arguments' as its function has parameters. |
| `FMA-06` | `PDDL_PredicateAtom` | error | Checks every entry of 'arguments' of a predicate atom (Mode B): a variable ('?x') is a parameter of an action whose precondition or effect contains the atom (typed pin, output pin without the suffix 'Out', or entry of 'parameters'), or a variable of a quantifier whose scope contains the atom; any other entry names a constant. In a formula that a problem holds in 'goal', an entry may also name an object. Atoms that no action, pin annotation or problem goal uses are not checked, because the transformation does not render them. The types of the arguments are not checked. |
| `FMA-07` | `PDDL_FunctionAtom` | error | Checks every entry of 'arguments' of a function atom (Mode B) in the same way as FMA-06: a parameter of an enclosing action, a variable of an enclosing quantifier, a constant, or in a problem goal an object. Atoms that nothing uses are not checked. The types of the arguments are not checked. |

## Numeric effects, comparisons and arithmetic (`FME`, 10 rules)

| ID | Context | Severity | Checks |
|---|---|---|---|
| `FME-01` | `PDDL_ConditionalEffect` | error | A conditional effect has a condition and an effect. |
| `FME-02` | `PDDL_Comparison` | error | A comparison has an operator, a left and a right operand. |
| `FME-03` | `PDDL_Assign` | error | An assign effect has a target function and a value. |
| `FME-04` | `PDDL_Increase` | error | An increase effect has a target function and a value. |
| `FME-05` | `PDDL_Decrease` | error | A decrease effect has a target function and a value. |
| `FME-06` | `PDDL_ScaleUp` | error | A scale-up effect has a target function and a factor. |
| `FME-07` | `PDDL_ScaleDown` | error | A scale-down effect has a target function and a divisor. |
| `FME-08` | `PDDL_ArithmeticExpression` | error | An arithmetic expression has an operator, a left and a right operand. |
| `FME-09` | `PDDL_ArithmeticExpression` | error | Both operands of an arithmetic expression are numeric terms (FunctionAtom, Literal or ArithmeticExpression). A predicate or a logical formula as operand gives invalid PDDL. |
| `FME-10` | `PDDL_Comparison` | error | Both operands of a comparison are numeric terms (FunctionAtom, Literal or ArithmeticExpression). A predicate or a logical formula as operand gives invalid PDDL. |

