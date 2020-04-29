# NanoIR

a little util function to help define s-expression style ir like: 

```
(let ([b #f])
    ((lambda (x)
       (if b 1 2))
     (lambda (x) x)))
```

if we want to define a ir so we can play some semantic on that, we need following predicate:

```
(begin
  (define (anf/if-term? term-match6)
    (match term-match6 ((? boolean?) #t) ((? anf-term?) #t) (else #f)))
  (define (anf/if-if-exp? g9)
    (match
     g9
     (`(if ,(? anf/if-aexp?) ,(? anf/if-cexp?) ,(? anf/if-cexp?)) #t)
     (else #f)))
  (define (anf/if-aexp? g10)
    (match
     g10
     ((? boolean?) #t)
     ((? symbol?) #t)
     ((? number?) #t)
     (`(lambda (,(? symbol?) ...) ,_) #t)
     (else #f)))
  (define (anf/if-cexp? g12)
    (match
     g12
     ((? anf/if-if-exp?) #t)
     ((? anf/if-aexp?) #t)
     (`(,(? anf/if-aexp?) ,(? anf/if-aexp?)) #t)
     (`(let ((,(? symbol?) ,(? anf/if-aexp?)) ...) ,(? anf/if-cexp?)) #t)
     (else #f)))
  (define (anf/if? ir5)
    (match
     ir5
     ((? anf/if-if-exp?) #t)
     ((? anf/if-aexp?) #t)
     ((? anf/if-cexp?) #t)
     (else #f))))
```

using `define-ir` , we can avoid writing this, just by using a EBNF-like style, and it will be macro expand to previous predicate

```
(define-ir anf
  (Term (x symbol?)
        (num number?))
  (Production
   (aexp
    ;; →
    (x
     num
     (lambda (x ...) _)))
   (cexp
    ;; →
    (aexp
     (aexp aexp)
     (let ([x aexp] ...)
       cexp)))))

(define-ir anf/if
  (Extend anf)
  (Term
   (+ bool boolean?))
  (Production
   (M aexp
      ((+ bool)))
   (+ if-exp
      ((if aexp cexp cexp)))
   (M cexp
      ((+ if-exp)))))

```

---------------
## How to use

Following macro is included here:

`define-ir`

```
(define-ir ir-name
  [(Extend parent-name)]
  (Term
   (meta-var tspec-predicate) ...)
  (Production
   ([mod] rules-name (rules ...))))
```
