# A Synced Effects Logic of Esterel

## For both Logical Correctness and Temporal Verification


Esterelisanimperativesynchronouslanguagethathasfound success in many safety-critical applications. Its precise semantics makes it natural for programming and reasoning. Existing techniques tackle either one of its main challenges: correctness checking or temporal verification. To resolve the issues simultaneously, we propose a new solution via a Hoare-style forward verifier and a term rewriting system (T.r.s) on Synced Effects. The first contribution is, by deploying a novel effects logic, the verifier computes the deterministic program behaviour via con- struction rules at the source level, defining program evaluation syntactically. As a second contribution, by avoiding the complex translation from LTL formulas to Esterel programs, our purely algebraic T.r.s efficiently checks temporal properties described by expressive Synced Effects. To demonstrate our method’s feasibility, we prototype this logic; prove its correctness; provide experimental results, and a number of case studies.


### To Compile:

```
git clone https://github.com/ForPaperReview/SyncedEffects_forReview.git
cd SyncedEffects_forReview
chmod 755 clean 
chmod 755 compile 
./compile
```

### Dependencies:

```
opam switch create 4.07.1
eval $(opam env)
sudo apt-get install menhir
sudo apt-get install z3
```

## Next, we display examples upon (1) Esterel Program Verification, (2) Entailments Checking, and (3) LTL to Effects Translation 

### (1) Program Verification Examples:
1. Overview Examples:
```
  module close:
  output CLOSE;

  /*@
  require {OPEN} 
  ensure {}.{CLOSE} 
  @*/

  pause; 
  emit CLOSE

  end module


  module manager:
  input BTN;
  output CLOSE;

  /*@
  require {} 
  ensure (({BTN}.{CLOSE})\/{})*
  @*/

  signal OPEN in 
    loop
      emit OPEN;
      present BTN 
        then run close
        else nothing
      end present;
      pause
    end loop
  end signal

  end module
```
Results by executing ./hip src/programs/0_verview.txt
```
========== Module: close ==========

(* Correctness Checking:  *)
({ /\ OPEN,!CLOSE,} . { /\ CLOSE,})
Logical Correct!

(* Temporal verification:   *)
[Pre  Condition] {OPEN,}
[Post Condition] ({} . {CLOSE,})
[Final  Effects] ({OPEN,!CLOSE,} . {CLOSE,})

[T.r.s: Verification for Post Condition]
----------------------------------------
({OPEN,!CLOSE,} . {CLOSE,}) |- ({} . {CLOSE,})
[Result] Succeed
[Verification Time: 1.6e-05 s]
 

* ({OPEN,!CLOSE,} . {CLOSE,}) |- ({} . {CLOSE,})   [UNFOLD]
* └── {CLOSE,} |- {CLOSE,}   [UNFOLD]
*     └── emp |- emp   [UNFOLD]



========== Module: manager ==========

(* Correctness Checking:  *)
({ /\ OPEN,} . ({ /\ !CLOSE,OPEN,})w)
Logical Correct!

(* Temporal verification:   *)
[Pre  Condition] {}
[Post Condition] (({BTN,} . {CLOSE,}) \/ {})*
[Final  Effects] ({OPEN,} . ({!CLOSE,OPEN,})w)

[T.r.s: Verification for Post Condition]
----------------------------------------
({OPEN,} . ({!CLOSE,OPEN,})w) |- (({BTN,} . {CLOSE,}) \/ {})*
[Result] Succeed
[Verification Time: 2.6e-05 s]
 

* ({OPEN,} . ({!CLOSE,OPEN,})w) |- (({BTN,} . {CLOSE,}) \/ {})*   [UNFOLD]
* └── ({!CLOSE,OPEN,})w |- (({BTN,} . {CLOSE,}) \/ {})*   [UNFOLD]
*     └── ({!CLOSE,OPEN,})w |- (({BTN,} . {CLOSE,}) \/ {})*   [Reoccur]
```

2. Loop:
```
  module testcase1:
  output A,B,C;

  /*@
  require {} 
  ensure {A, B} . ({B, C})*
  @*/

  emit A;
  loop 
    emit B;
    pause;
    emit C
  end loop 

  end module
```
Results by executing ./hip src/programs/1_loop.txt.
```
========== Module: testcase1 ==========

(* Correctness Checking:  *)
({ /\ A,B,} . ({ /\ C,B,})w)
Logical Correct!

(* Temporal verification:   *)
[Pre  Condition] {}
[Post Condition] ({A,B,} . ({B,C,})*)
[Final  Effects] ({A,B,} . ({C,B,})w)

[T.r.s: Verification for Post Condition]
----------------------------------------
({A,B,} . ({C,B,})w) |- ({A,B,} . ({B,C,})*)
[Result] Succeed
[Verification Time: 2.4e-05 s]
 

* ({A,B,} . ({C,B,})w) |- ({A,B,} . ({B,C,})*)   [UNFOLD]
* └── ({C,B,})w |- ({B,C,})*   [UNFOLD]
*     └── ({C,B,})w |- ({B,C,})*   [Reoccur]
```

3. Present:
```
  module testcase7:
  output A,B, C;
  /*@
  require {} 
  ensure ({S2 })*
  @*/

 signal S2 in 
 signal S1 in
 signal S  in 
 present S then emit S1 else emit S2
 end present
 end signal
 end signal
 end signal

  end module
```
Results by executing ./hip src/programs/7_present.txt.
```
========== Module: testcase7 ==========

(* Correctness Checking:  *)
{!S, /\ !S1,!S,!A,!B,!C,S2,}
Logical Correct!

(* Temporal verification:   *)
[Pre  Condition] {}
[Post Condition] ({S2,})*
[Final  Effects] {!S1,!S,!A,!B,!C,S2,}

[T.r.s: Verification for Post Condition]
----------------------------------------
{!S1,!S,!A,!B,!C,S2,} |- ({S2,})*
[Result] Succeed
[Verification Time: 1.7e-05 s]
 

* {!S1,!S,!A,!B,!C,S2,} |- ({S2,})*   [UNFOLD]
* └── emp |- ({S2,})*   [UNFOLD]
```


### (2) Entailments Checking Examples:
1 ./sleek src/effects/Disjunction_both.ee
```
====================================
[A;B] + [C] + [B;D] |- [A;D] + [B]
[Result] Fail
[Verification Time: 5.6e-05 s]
 

* [A;B] + [C] + [B;D] |- [A;D] + [B]
* └── (-[A;B])[A;B] + [C] + [B;D] |- [A;D] + [B]   [UNFOLD]
*     ├── (-[C])[A;B] + [C] + [B;D] |- [A;D] + [B]   [UNFOLD]
*     │   ├── (-[B;D])[A;B] + [C] + [B;D] |- [A;D] + [B]   [UNFOLD]
*     │   │   └── Emp |- Emp   [PROVE]
*     │   └── Emp |- _|_   [DISPROVE]
*     └── Emp |- Emp   [PROVE]

====================================
[A;B] + [C] + [B;D] |- [A;D] + [B] + [C]
[Result] Succeed
[Verification Time: 8.2e-05 s]
 

* [A;B] + [C] + [B;D] |- [A;D] + [B] + [C]
* └── (-[A;B])[A;B] + [C] + [B;D] |- [A;D] + [B] + [C]   [UNFOLD]
*     ├── (-[C])[A;B] + [C] + [B;D] |- [A;D] + [B] + [C]   [UNFOLD]
*     │   ├── (-[B;D])[A;B] + [C] + [B;D] |- [A;D] + [B] + [C]   [UNFOLD]
*     │   │   └── Emp |- Emp   [PROVE]
*     │   └── Emp |- Emp   [PROVE]
*     └── Emp |- Emp   [PROVE]

====================================
[A;C] + ([B;D])^* |- (_)^* + [A]
[Result] Succeed
[Verification Time: 5.4e-05 s]
 

* [A;C] + ([B;D])^* |- (_)^* + [A]
* └── (-[A;C])[A;C] + ([B;D])^* |- (_)^* + [A]   [UNFOLD]
*     └── (-[B;D])[A;C] + ([B;D])^* |- (_)^* + [A]   [UNFOLD]
*         └── (-[B;D])([B;D])^* |- (_)^*   [UNFOLD]
*             └── ([B;D])^* |- (_)^*   [PROVE]

====================================
[D;B] + [A;B].[C;D] |- [A].[C] + [B]
[Result] Succeed
[Verification Time: 6.6e-05 s]
 

* [D;B] + [A;B].[C;D] |- [A].[C] + [B]
* └── (-[D;B])[D;B] + [A;B].[C;D] |- [A].[C] + [B]   [UNFOLD]
*     └── (-[A;B])[D;B] + [A;B].[C;D] |- [A].[C] + [B]   [UNFOLD]
*         └── (-[C;D])[C;D] |- [C] + Emp   [UNFOLD]
*             └── Emp |- Emp   [PROVE]

====================================
[A;B] + [C;D] + [E] |- [B] + (_)^*
[Result] Succeed
[Verification Time: 6.8e-05 s]
 

* [A;B] + [C;D] + [E] |- [B] + (_)^*
* └── (-[A;B])[A;B] + [C;D] + [E] |- [B] + (_)^*   [UNFOLD]
*     ├── (-[C;D])[A;B] + [C;D] + [E] |- [B] + (_)^*   [UNFOLD]
*     │   ├── (-[E])[A;B] + [C;D] + [E] |- [B] + (_)^*   [UNFOLD]
*     │   │   └── Emp |- (_)^*   [PROVE]
*     │   └── Emp |- (_)^*   [PROVE]
*     └── Emp |- Emp + (_)^*   [PROVE]
```


### (3) LTL to Effects Translator Examples:

./ltl src/ltl/Traffic_light.ltl 

```
====================================
([] Green)

[Translated to Effects] ===>
 
([ Green])^* 

====================================
(<> Green)

[Translated to Effects] ===>
 
(_)^* . [ Green] 

====================================
(<> Red)

[Translated to Effects] ===>
 
(_)^* . [ Red] 

====================================
(<> Yellow)

[Translated to Effects] ===>
 
(_)^* . [ Yellow] 

====================================
(<> ([] Green))

[Translated to Effects] ===>
 
(_)^* . ([ Green])^* 

====================================
([] (<> Red))

[Translated to Effects] ===>
 
((_)^* . [ Red])^* 

====================================
((<> Red) -> (! (Green U Red)))

[Translated to Effects] ===>
 
(!(_)^* . [ Red])
(!([ Green])^* . [ Red]) 

====================================
([] (Red -> (! (XGreen))))

[Translated to Effects] ===>
 
((![ Red]))^*
((!_ . [ Green]))^* 

====================================
([] (Red -> (<> Green)))

[Translated to Effects] ===>
 
((![ Red]))^*
((_)^* . [ Green])^* 

====================================
([] (Red -> ([] (! Green))))

[Translated to Effects] ===>
 
((![ Red]))^*
(((![ Green]))^*)^* 

====================================
([] (Red -> (! (Green U Yellow))))

[Translated to Effects] ===>
 
((![ Red]))^*
((!([ Green])^* . [ Yellow]))^* 

====================================
([] (Red -> (X(Red U Yellow))))

[Translated to Effects] ===>
 
((![ Red]))^*
(_ . ([ Red])^* . [ Yellow])^* 

====================================
([] (Red -> (X(Red U (X(Yellow U Green))))))

[Translated to Effects] ===>
 
((![ Red]))^*
(_ . ([ Red])^* . _ . ([ Yellow])^* . [ Green])^* 
```

### To Clean:

``` 
./clean
```

 
