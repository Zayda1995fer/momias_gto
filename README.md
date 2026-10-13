# Momias de Guanajuato: La Leyenda

Juego de plataformas hecho en Flutter y construido con TDD (Desarrollo Guiado por Pruebas). Un explorador cruza los callejones de Guanajuato saltando entre plataformas, esquivando o derrotando momias y recolectando calaveritas de azúcar hasta llegar al Callejón del Beso.

Toda la física y las reglas del juego (gravedad, salto, colisiones, patrullaje de las momias, combate, puntaje y vidas) se programaron como Dart puro y se probaron antes de escribir la interfaz.

**Materia:** Desarrollo Móvil Integral
**Alumno(a):** Zayda Fernanda Vargas Vargas
**Fecha:** octubre 12 del 2026

## Mecánicas del juego

- El personaje cae por gravedad y aterriza sobre plataformas.
- Botones en pantalla lo mueven a la izquierda o derecha y lo hacen saltar.
- Las momias patrullan entre dos límites y dan media vuelta al llegar a ellos.
- Saltar encima de una momia la derrota; chocar de lado con ella quita una vida.
- Cada calaverita recolectada suma 50 puntos.
- Llegar al Callejón del Beso gana la partida; quedarse sin vidas la pierde.

## Estructura del proyecto

```
lib/
  logica/
    rectangulo.dart       colisión AABB
    personaje.dart
    momia.dart
    calaverita.dart
    motor_fisica.dart     gravedad, salto, aterrizaje, límites, meta
    motor_combate.dart    pisar a la momia vs. ser golpeado
    estado_juego.dart
    motor_juego.dart      orquesta todas las reglas en cada cuadro
  presentation/
    juego_screen.dart     cámara, dibujo y controles
  main.dart
test/
  rectangulo_test.dart
  motor_fisica_test.dart
  momia_test.dart
  motor_combate_test.dart
  motor_juego_test.dart
```

## Cómo ejecutar

```bash
flutter test   # corre las 23 pruebas
flutter run    # abre el juego
```

## Pruebas

Las capturas de evidencia se guardan en la carpeta `evidencias/`.

### Ronda 1. Colisión entre rectángulos (AABB)

**Archivo:** `test/rectangulo_test.dart`

- **`dos rectangulos que se traslapan se detectan como interseccion`**: dos rectángulos que se enciman deben detectarse como colisión.
- **`dos rectangulos separados no se intersectan`**: dos rectángulos lejos uno del otro no deben chocar.

Esta colisión es la base de todo el juego: se usa para el piso, las momias y las calaveritas.

<img width="631" height="215" alt="ronda1" src="https://github.com/user-attachments/assets/f4ae7aef-4127-4ee6-8bfb-dc40e5a14113" />


### Ronda 2. Gravedad y salto

**Archivo:** `test/motor_fisica_test.dart`

- **`aplicarGravedad aumenta la velocidad vertical y mueve al personaje hacia abajo`**: en cada cuadro la gravedad acelera la caída del personaje.
- **`saltar solo funciona si el personaje esta en el suelo`**: desde el suelo, el salto da un impulso hacia arriba.
- **`saltar no hace nada si el personaje ya esta en el aire`**: evita el doble salto infinito.
<img width="637" height="145" alt="ronda2" src="https://github.com/user-attachments/assets/c243d970-f428-4282-8fbc-2c764a7cb5a3" />


### Ronda 3. Aterrizar sobre una plataforma

**Archivo:** `test/motor_fisica_test.dart`

- **`el personaje aterriza sobre una plataforma al caer sobre ella`**: si en el cuadro anterior el pie estaba arriba de la plataforma y ahora la cruzó, el personaje queda parado exactamente encima.
- **`el personaje no aterriza si todavia esta muy por encima de la plataforma`**: si aún no la alcanza, sigue cayendo.

**Corrección al manual:** las posiciones originales de la primera prueba (`y: 90`, `yAnterior: 85`) ponían el pie del personaje dentro de la plataforma desde el cuadro anterior, por lo que nunca aterrizaba. Se cambiaron a `y: 70` y `yAnterior: 65`. También se agregó el `import 'rectangulo.dart'` que faltaba en `motor_fisica.dart`.

<img width="625" height="132" alt="ronda3" src="https://github.com/user-attachments/assets/3296b6b7-7a99-4360-b15e-379ca234c940" />


### Ronda 4. Movimiento horizontal con límites del nivel

**Archivo:** `test/motor_fisica_test.dart`

- **`moverHorizontal desplaza al personaje segun el delta recibido`**: el personaje avanza la distancia indicada.
- **`moverHorizontal no deja salir al personaje por el borde izquierdo`**: no puede pasar de la posición 0.
- **`moverHorizontal no deja salir al personaje por el borde derecho`**: no puede salirse por el final del nivel.

<img width="621" height="122" alt="ronda4" src="https://github.com/user-attachments/assets/7a9e3572-575c-493c-aa9d-d8de9d8e8821" />


### Ronda 5. La momia patrulla entre dos límites

**Archivo:** `test/momia_test.dart`

- **`la momia avanza en linea recta mientras no llegue a un limite`**: se mueve según su velocidad.
- **`la momia invierte direccion al llegar al limite derecho`**: al tocar su límite da media vuelta.
- **`una momia derrotada no se mueve`**: las momias derrotadas se quedan quietas.

<img width="632" height="137" alt="ronda5" src="https://github.com/user-attachments/assets/cdab09cc-6a1f-4863-85bc-2305e74d0ee5" />


### Ronda 6. Combate

**Archivo:** `test/motor_combate_test.dart`

- **`caer sobre la mitad superior de una momia la derrota`**: si el personaje va cayendo y su pie está en la mitad de arriba de la momia, la derrota y rebota.
- **`chocar de lado con una momia golpea al jugador`**: cualquier otro choque lastima al jugador.
- **`sin interseccion no pasa nada`**: si no se tocan, no hay efecto.

**Corrección al manual:** en la primera prueba el personaje estaba en `y: 68`, con el pie exactamente en el borde de la momia. Como la colisión AABB no cuenta los bordes que solo se tocan, no detectaba el choque. Se cambió a `y: 70`.

<img width="617" height="135" alt="ronda6" src="https://github.com/user-attachments/assets/7de2190c-54d4-4e31-9758-f211681ed0a2" />


### Ronda 7. Recolectar calaveritas

**Archivo:** `test/motor_juego_test.dart`

- **`recolectar una calaverita suma puntos y la marca como recolectada`**: tocar una calaverita da 50 puntos.
- **`una calaverita ya recolectada no vuelve a sumar puntos`**: cada calaverita cuenta una sola vez.
<img width="626" height="132" alt="ronda7" src="https://github.com/user-attachments/assets/99d93235-815a-4825-b97b-b005d0ea7a8b" />


### Ronda 8. Detectar la meta (el Callejón del Beso)

**Archivo:** `test/motor_fisica_test.dart`

- **`haLlegadoALaMeta es verdadero cuando el personaje alcanza la posicion`**: llegar justo a la meta ya cuenta.
- **`haLlegadoALaMeta es falso si aun no llega`**.

<img width="965" height="455" alt="ronda8" src="https://github.com/user-attachments/assets/851579a1-5b6a-4952-8ceb-2a96fac7da52" />


### Ronda 9. Vidas y Game Over (rojo)

**Archivo:** `test/motor_juego_test.dart`

- **`al chocar lateralmente con una momia se pierde una vida y el personaje regresa al inicio`**: de 3 vidas pasa a 2 y vuelve a su posición inicial.
- **`perder la ultima vida cambia el estado a perdido`**: con la última vida, el juego termina.

Estas pruebas se escribieron antes de que existiera el motor del juego; la captura muestra el rojo.

<img width="921" height="907" alt="ronda9" src="https://github.com/user-attachments/assets/953d56e6-44d2-4268-8463-6b035f04575c" />


### Ronda 10. El motor que integra todo (verde)

Se creó `EstadoJuego` y la clase completa `MotorJuego`, cuyo método `mover()` ejecuta en cada cuadro, siempre en el mismo orden: física, momias, calaveritas y meta. Con esto pasan las pruebas de la ronda 9.

<img width="626" height="170" alt="ronda10" src="https://github.com/user-attachments/assets/bff4c7d1-4e2b-4c2d-86a9-2fc7ff127773" />


### Prueba de integración. Ganar llegando a la meta

**Archivo:** `test/motor_juego_test.dart`

- **`llegar a la meta cambia el estado a ganado`**: con una sola llamada a `mover()`, el personaje avanza, la gravedad y el piso actúan, y al cruzar la meta el estado cambia a `ganado`.

![Evidencia integración](evidencias/integracion.png)

## Resumen de pruebas

| # | Ronda | Prueba | 
|---|-------|--------|
| 1 | 1 | Rectángulos que se traslapan chocan | 
| 2 | 1 | Rectángulos separados no chocan | 
| 3 | 2 | La gravedad acelera la caída | 
| 4 | 2 | Se puede saltar desde el suelo | 
| 5 | 2 | No se puede saltar en el aire | 
| 6 | 3 | Aterriza al caer sobre la plataforma | 
| 7 | 3 | No aterriza si aún está arriba | 
| 8 | 4 | Se mueve según el delta | 
| 9 | 4 | No sale por el borde izquierdo | 
| 10 | 4 | No sale por el borde derecho | 
| 11 | 5 | La momia avanza | 
| 12 | 5 | La momia da media vuelta en el límite | 
| 13 | 5 | La momia derrotada no se mueve | 
| 14 | 6 | Pisar a la momia la derrota | 
| 15 | 6 | Chocar de lado golpea al jugador | 
| 16 | 6 | Sin contacto no pasa nada | 
| 17 | 7 | La calaverita suma 50 puntos | 
| 18 | 7 | La calaverita no suma dos veces | 
| 19 | 8 | Llegar a la meta | 
| 20 | 8 | Aún no llega a la meta | 
| 21 | 9 | Perder una vida y regresar al inicio | 
| 22 | 9 | Perder la última vida termina el juego | 
| 23 | Integración | Ganar llegando a la meta | 

**Resultado final de `flutter test`:**


<img width="626" height="142" alt="todaslas pruebas" src="https://github.com/user-attachments/assets/9dc141dd-b658-4e71-b7de-55e83a31f15d" />

## Evidencia del juego
<img width="1291" height="986" alt="iniciojuego" src="https://github.com/user-attachments/assets/81bccba0-b47f-4777-813f-a24c74a3da8d" />
<img width="1267" height="936" alt="juegoFn" src="https://github.com/user-attachments/assets/623bfd45-8ea8-450a-95cc-9bda677c5cf4" />

