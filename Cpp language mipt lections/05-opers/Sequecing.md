Выражения, разделённый точкой с запятой состоят в 
**отношениях последования** `sequenced-after` и `sequenced-before`:
```cpp
foo(); bar();
// foo sequenced before bar
```

Но увы, вызов функции не определяет `sequencing`:
```cpp
buzz(foo(), bar());
// no sequencing between foo and bar
```

Важно это потому, что `unsequenced modification` - это просто `UB`:
```cpp
y = x++ + x++; 
// operator++ and operator++ unsequenced - UB
```

