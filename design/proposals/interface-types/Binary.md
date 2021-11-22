# Interface Types Binary Format

## Convention

```
T? ::= 0x00     => ϵ
     | 0x01 t:T => t
```

## Interface Types

Extending [`deftype`](../module-linking/Binary.md#import-definitions) and
[`type`](../module-linking/Binary.md#type-definitions) as defined by
module-linking:
```
deftype            ::= ...
                     | 0x06 i:<typeidx>                      => type-index-space[i]
                                                                (must be <adapter-functype>)
type               ::= ...
                     | aft:<adapter-functype>                => aft
                     | cit:<compound-intertype>              => cit
adapter-functype   ::= 0x7c p*:<named-type>* r?:<intertype>? => (adapter func (param p)* (result r)?)
named-type         ::= n:<name> t:<intertype>                => n t
compound-intertype ::= 0x7b t:<intertype>                    => (list t)
                     | 0x7a nt*:vec(<named-type>)            => (record (field nt)*)
                     | 0x79 nmt*:vec(<named-maybe-type>)     => (variant (case nmt)*)
                     | 0x78 t*:vec(<intertype>)              => (tuple t*)
                     | 0x77 name*:vec(<name>)                => (flags name*)
                     | 0x76 tag*:vec(<name>)                 => (enum tag*)
                     | 0x75 t*:vec(<intertype>)              => (union t*)
                     | 0x74 t:<intertype>                    => (option t)
                     | 0x73 t?:<intertype>? u?:<intertype>?  => (expected t? (error u)?)
                     | 0x72 n:<name> t:<intertype>           => (named n t)
named-maybe-type   ::= n:<name> t?:<intertype>               => n t?
intertype          ::= i:<typeidx>                           => type-index-space[i]
                                                                (must be <compound-intertype>)
                     | 0x71                                  => bool
                     | 0x70                                  => s8
                     | 0x6f                                  => u8
                     | 0x6e                                  => s16
                     | 0x6d                                  => u16
                     | 0x6c                                  => s32
                     | 0x6b                                  => u32
                     | 0x6a                                  => s64
                     | 0x69                                  => u64
                     | 0x68                                  => float32
                     | 0x67                                  => float64
                     | 0x66                                  => char
                     | 0x65                                  => string
```

## Canonical Adapter Function Definitions

Extending [`section`](../module-linking/Binary.md#module-definitions),
[`def-ref`](../module-linking/Binary.md#instance-definitions) and
[`alias`](../module-linking/Binary.md#alias-definitions) as defined by
module-linking:
```
section           ::= ...
                    | f*:section_7(vec(<func>))                       => f*
                    | f*:section_8(vec(<adapter-func>))               => f*
func              ::= i:<typeidx> body:<func-body>                    => (func type-index-space[i] body)
                                                                         where i refers to a <core:functype>
func-body         ::= 0x00 f:<adapter-funcidx> opts*:vec(<canon-opt>) => (canon.lower $f opts*)
adapter-func      ::= i:<typeidx> body:<adapter-func-body>            => (adapter func type-index-space[i] body)
                                                                         where i refers to an <adapter-functype>
adapter-func-body ::= 0x00 f:<funcidx> opts*:vec(<canon-opt>)         => (canon.lift $f opts*)
canon-opt         ::= 0x00                                            => string=utf8
                    | 0x01                                            => string=utf16
                    | 0x02                                            => string=compact-utf16
                    | 0x03 m:<memidx>                                 => (memory $m)
                    | 0x04 f:<funcidx>                                => (realloc $f)
                    | 0x05 f:<funcidx>                                => (free $f)
def-ref           ::= ...
                    | 0x06 x:<adapter-funcidx>                       => (adapter func x)
alias             ::= ...
                    | 0x00 i:<instanceidx> nm:<name> 0x06            => (alias i nm (adapter func))
```

## Initialization Values

Extending [`section`](../module-linking/Binary.md#module-definitions), 
[`deftype`](../module-linking/Binary.md#import-definitions),
[`def-ref`](../module-linking/Binary.md#instance-definitions) and
[`alias`](../module-linking/Binary.md#alias-definitions) again:
```
section  ::= ...
           | s:section_9(<start>)                      => s
start    ::= f:<adapter-funcidx> u*:vec(<initval-idx>) => (start $f (value u)*)
deftype  ::= ...
           | 0x07 t:<intertype>                        => (value t)
def-ref  ::= ...
           | 0x07 x:<initvalidx>                       => (value x)
alias    ::= ...
           | 0x00 i:<instanceidx> nm:<name> 0x07       => (alias i nm (value))
```

## Component Definitions

Reusing [top-level productions defined by module-linking](../module-linking/Binary.md#module-definitions):
```
component-version  ::= 0x0a 0x00 0x02 0x00
component-preamble ::= <magic> <component-version>
component          ::= <component-preamble> s*:<section>* => (component flatten(s*))
```
