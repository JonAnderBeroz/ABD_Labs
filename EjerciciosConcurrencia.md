# Ejercicios Concurrencia

1.¿Cuál(es) de las siguientes planificaciones es(son) serializable(s)?

a) r1(x) r2(y) r1(z) r3(z) r2(x) r1(y) (Serializable)

b) r1(x) w2(y) r1(z) r3(z) w2(x) r1(y)

c) r1(x) w2(y) r1(z) r3(z) w1(x) r2(y) (Serializable)

    t1 => read X read Z write X
    t2 => write Y read Y
    t3 => read >

d) r1(x) r2(y) r1(z) r3(z) w1(x) w2(y) (Serializable)

e) r1(x) r2(y) w2(x) w3(x) w3(y) r1(y)  (Serializable)

f) w1(x) r2(y) r1(z) r3(z) r1(x) w2(y) (Serializable)

g) r1(z) w2(x) r2(z) r2(y) w1(x) w3(z) w1(y) r3(x) (Serializable)

2.Obtén cuatro ordenaciones posibles equivalentes por conflicto correspondientes al grafo de serialización de las figuras siguientes: 

    1. 
    - T1 T2 T3 T5 T6 T4 T7
    - T1 T3 T5 T6 T7 T2 T4
    - T1 T3 T5 T6 T2 T4 T7
    - T1 T3 T2 T5 T6 T4 T7
    2. En esta caso no seria posible serializarlo en conflicto ya que para poder hacer T6 dependemos de T3 T5 y T2, pero a su vez T2 depende de T7, que a su vez depende de T6.

15.Dada la planificación que usa la técnica de validación que muestra la figura, donde las líneas horizontales muestran el transcurso del tiempo, las líneas gruesas verticales el final de la transacción, la línea punteada indican la fase de validación y las posteriores líneas verticales la fase de escritura.

- El protocolo de marcas de tiempo sin multiversión.
MT(T1) = 1
MT(T2) = 2
MT(T3) = 3