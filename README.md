# A Synced Effects Logic of Esterel

## For both Logical Correctness and Temporal Verification


Esterel is an imperative style language that has found success in many safety-critical applications. Its precise semantics makes it natural for programming and reasoning. Existing techniques tackle either one of its main challenges: correctness checking or temporal verification. To resolve the issues simultaneously, we propose a new solution via a Hoare-style forward verifier and a term rewriting system (T.r.s) on Synced Effects. The first contribution is the novel effects logic computes the deterministic program behaviour via construction rules at the source level, defining program evaluation syntactically. As a second contribution, by avoiding the complex translation from temporal properties to Esterel programs, our purely algebraic T.r.s efficiently checks the temporal properties described by the Synced Effects. We prototype this logic on top of the HIP/SLEEK system and show our method’s feasibility using a number of case studies.

### To Compile:

```
git clone https://github.com/APLAS-20-Sub-14/SyncedEffects.git
cd SyncedEffects
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


### Entailments Checking Examples:
```
/*
    ensure ([S2 ]^*)
*/

(signal S2 (signal S1 
(signal S
  (present S emit S1 emit S2))
))
```
Execution results: (./hip src/programs/APLAS20_fig6.txt )
```
<<<<< Logical Correctness Checking >>>>>
=========================
Logical correct! 
Forward Result = 
[ S2  ]

 <<<<< Temporal Verification >>>>>
====================================
[S2] |- ([S2])^*
[Result] Succeed
[Verification Time: 2.1e-05 s]
 

* [S2] |- ([S2])^*
* └── (-[S2])[S2] |- ([S2])^*   [UNFOLD]
*     └── Emp |- ([S2])^*   [PROVE]
```


```
/*
    ensure [ A, E  ] . [  B, C,   F ] . [ G] \/ (_^*) . [G]
*/

(signal A (signal B (signal C (signal D (signal E (signal F (signal G 
  (
  (emit A;
    (pause;
      (emit B;
        emit C
      )))
  ||
  (emit E;
    (pause;
      (emit F;
        (pause;
          emit G
        )))))
)))))))
```
Execution results: (./hip src/programs/APLAS20_fig7.txt)
```
<<<<< Logical Correctness Checking >>>>>
=========================
Logical correct! 
Forward Result = 
[ A    E  ] . [  B C   F ] . [       G]

 <<<<< Temporal Verification >>>>>
====================================
[A;E].[B;C;F].[G] |- [A;E].[B;C;F].[G] + (_)^*.[G]
[Result] Succeed
[Verification Time: 9e-05 s]
 

* [A;E].[B;C;F].[G] |- [A;E].[B;C;F].[G] + (_)^*.[G]
* └── (-[A;E])[A;E].[B;C;F].[G] |- [A;E].[B;C;F].[G] + (_)^*.[G]   [UNFOLD]
*     └── (-[B;C;F])[B;C;F].[G] |- [B;C;F].[G] + (_)^*.[G]   [UNFOLD]
*         └── (-[G])[G] |- [G] + (_)^*.[G]   [UNFOLD]
*             └── Emp |- (_)^*.[G] + Emp   [PROVE]
```



### Examples:

Entailments Checking 

```
./sleek src/effects/test1.txt 
```

Program Verification

```
./hip src/programs/fig5.txt
```

LTL to Effects Translator

```
./ltl src/ltl/Traffic_light.ltl 
```

### To Clean:

``` 
./clean
```

 
