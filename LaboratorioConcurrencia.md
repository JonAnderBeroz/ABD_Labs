# Ejercicios Concurrencia

1. ¿Cuál(es) de las siguientes planificaciones es(son) serializable(s)?

a) r1(x) r2(y) r1(z) r3(z) r2(x) r1(y) (Serializable)

b) r1(x) w2(y) r1(z) r3(z) w2(x) r1(y)

c) r1(x) w2(y) r1(z) r3(z) w1(x) r2(y) (Serializable)

    t1 => read X read Z write X
    t2 => write Y read Y
    t3 => read >

d) r1(x) r2(y) r1(z) r3(z) w1(x) w2(y) (Serializable)

e) r1(x) r2(y) w2(x) w3(x) w3(y) r1(y)  (Serializable)

f) w1(x) r2(y) r1(z) r3(z) r1(x) w2(y) (Serializable)

g) r1(z) w2(x) r2(z) r2(y) w1(x) w3(z) w1(y) r3(x)

2. Obtén cuatro ordenaciones posibles equivalentes por conflicto correspondientes al grafo de serialización de las figuras siguientes: 
- T1 => T2 => T4 , T1 => T3 => T5 => T6 => T7, T1 => T3 => T5 => T6 => T4, T1 => T3 => T6 => T7
- T1 => T2 => T4, T1 => T2  => T6 => T7 => T2 => T4, T1 => T3 => T5 => T6 => T7 => T2 => T4, T1 => T3 =>     T6 => T7 => T2 => T4

