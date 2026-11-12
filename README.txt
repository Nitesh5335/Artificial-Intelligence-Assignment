AI Assignment - Classical Reasoning Agent
=========================================

Files
-----
reasoning_agent.ipynb  complete implementation, sanity tests, reasoning traces,
                       public-dataset experiments and submission generation
submission.csv         predictions (columns: id, flag)
report.pdf             short report (representation, algorithms, experiments, traces)
README.txt             this file

Requirements
------------
Python 3.8+, pandas.   Optional for the public-dataset experiments:
    pip install datasets
    huggingface-cli login      (yale-nlp/FOLIO is a gated dataset)
The reasoning engine itself uses only the Python standard library.

How to run on the test set
--------------------------
1. Open reasoning_agent.ipynb.
2. In the last section set   TEST_PATH = '<path to test file>'
   (csv / tsv / json / jsonl / parquet / xlsx).
   Columns are detected automatically:
     id         : id, example_id, idx ...
     knowledge  : context, premises-FOL, premises, logic, kb ...
     query      : statement, conclusion-FOL, query, question ...
     gold label : flag, label, answer ...   (optional; used to print accuracy)
3. Run all cells.  The notebook prints the final accuracy (if gold labels exist)
   and the total inference time, and writes submission.csv.
   To skip the public experiments set RUN_PUBLIC_EXPERIMENTS = False.

Input logic format
------------------
One formula per line (or separated by ';' or a sentence-final '.'), or a list of
formulas.  Supported notations:
   ∀x (Young(x) → Smart(x))          ∃x P(x)     ¬ ∧ ∨ → ↔ ⊕  = ≠
   forall x. young(x) -> smart(x)    exists x.   ~ ! not  & and  | or  => <-> iff xor
   smart(X) :- young(X), kind(X).    (Prolog style)
Free variables x, y, z, u, v, w (and X, Y, Z ...) are universally quantified.
Predicate and constant names are case-insensitive (CONFIG['normalize_case']).

Semantics
---------
Output True  iff  KB |= query  (KB ∧ ¬query is unsatisfiable).
Output False otherwise (the KB entails ¬query, or the query is undetermined).
For closed-world data set CONFIG['closed_world'] = True (negation as failure).

Reasoning methods (chosen automatically per query)
--------------------------------------------------
- Definite Horn KB          : semi-naive forward chaining (least Herbrand model)
- Function-free FOL         : Herbrand grounding + DPLL (complete decision procedure)
- FOL with function symbols : depth-bounded grounding + DPLL, then resolution
                              refutation (set of support, best-first selection)
All steps use unification, CNF conversion and Skolemisation implemented in the notebook.
No generative model is used for the reasoning.
